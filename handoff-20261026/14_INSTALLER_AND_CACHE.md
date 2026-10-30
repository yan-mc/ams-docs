# 14. 下载缓存重复与双卸载器问题（v1.0.0 发现 → v1.0.1 修复）

日期：2026-10-30  
范围：`F:\AMS-Studio`（v1.0.0 实际安装）→ 源码修复 → v1.0.1 发布（feed 2053）  
证据日志：`_接管审计/logs/cleanup-downloads-20261030.log`

---

## 1. 现象

用户报告安装后磁盘占用异常大。实测 v1.0.0 安装：

| 项 | 数值 |
|---|---|
| 安装总大小 | **59.31 GB** / 128,380 文件 |
| `data/components/downloads` | **35,409.4 MB** / 26 文件（纯下载暂存区） |
| 其中分卷文件 `*-part-*.bin` | ~17.3 GB |
| 其中合并后的 `*.zip` | ~18.1 GB |
| 实际安装后的运行数据（kernel/runtime/assets/tools…） | ~24.7 GB |

另外安装根目录有两个**字节完全相同**的卸载器：`AMS-Studio-Uninstall.exe` 与 `uninstall.exe`（54,692 B，sha256 `8D844270…540B`）。

## 2. 根因

### 2.1 下载暂存区从不清空（重复约 2 倍）

`ams-launcher/src-tauri/src/lib.rs`：

1. `download_parts_to()`：把 HuggingFace 分卷逐个下载到
   `downloads/<id>-<ver>-part-N.bin`，校验 sha 后追加写入 `downloads/<id>-<ver>.zip`。
   **分卷在合并完成后没有删除** → 分卷 + zip 双份共存（rvc 6.87G+6.87G、pymss 3.7G+3.7G、tts-train 6.63G+6.63G…）。
2. `install_component_blocking()`：下载/合并出 zip → 校验 → `extract_zip()` 解包到目标目录。
   **解包成功后没有删除 zip（也没有清理分卷）** → 所有组件安装包永久留在 downloads。
3. 没有启动时/安装后的清理逻辑，安装中断也会永久留下残片。

合计：分卷（~17.3GB，合并后即无用） + 安装包（~18.1GB，解包后即无用） = **35.4GB 冗余**，在 59.31GB 安装里占 60%。

### 2.2 两个卸载器

`installer/ams-setup.nsi` 原逻辑：

```nsis
WriteUninstaller "$INSTDIR\AMS-Studio-Uninstall.exe"
CopyFiles "$INSTDIR\AMS-Studio-Uninstall.exe" "$INSTDIR\uninstall.exe"
```

- launcher 的"安装检测"（`installed_data_dir`）和"打开卸载程序"（`open_uninstaller`）都要求 `uninstall.exe`；
- 但 NSIS 卸载器被写成了 `AMS-Studio-Uninstall.exe`，于是用 `CopyFiles` 复制出第二份 `uninstall.exe`；
- 两个文件字节相同、功能相同，只是名字不同 → 用户看到"两个卸载软件"。

## 3. 修复（v1.0.1）

源码提交：`ams-launcher` **7602359**（launcher 1.0.1）。

| 修复 | 实现 |
|---|---|
| 分卷即用即删 | `download_parts_to()` 每卷合并后立即 `fs::remove_file(part_path)` |
| 安装包解包即删 | `install_component_blocking()` 解包+配置成功后删除 zip 并清理 downloads 残留 |
| 安装收尾清理 | GUI/CLI 安装循环结束后 `cleanup_downloads_dir()` 全量清扫 |
| 启动自愈 | launcher 启动时清扫 `components/downloads`（覆盖上次中断残留） |
| 手动清理命令 | 新增 `ams-launcher.exe --cleanup-downloads`（写日志 `logs/launcher-cleanup-downloads.log`） |
| 单卸载器 | setup 只 `WriteUninstaller "$INSTDIR\uninstall.exe"`；注册表 `UninstallString` 指向它；安装时删除旧版遗留的 `AMS-Studio-Uninstall.exe`；卸载段兼容清理旧文件 |

> 注：`downloads` 是纯暂存区——组件解包后不再读取；删除安全。用户数据（voices/datasets/projects/models/config/logs）不受影响。

## 4. 验证

### 4.1 全新安装 + 小规模真实组件安装（launcher 1.0.1）

用本地测试 feed（seq 2099，签名有效，4 个组件）安装到 `F:\_ams-fix-test\app`：

- setup 1.0.1 安装：目录只有 `ams-launcher.exe` + **单个 `uninstall.exe`**；`AMS-Studio-Uninstall.exe` 不存在；注册表 `UninstallString="...\uninstall.exe"`、version 1.0.1。
- 组件安装（kernel 388MB、catalog、gateway、plugin 拆成 2 个分卷）：**9.8 秒完成，4/4 ok=true**；
- 安装后 `data/components/downloads`：**0 文件**；无任何 `-part-`/`.zip` 残留；
- `--cleanup-downloads`：放入 1MB 假文件后执行 → 被清理，退出码 0；
- 升级兼容：手工放回旧版 `AMS-Studio-Uninstall.exe` 后重装 1.0.1 setup → 旧文件被删除，仅剩 `uninstall.exe`。

### 4.2 当前 v1.0.0 安装的立即清理

- 逐条删除 `F:\AMS-Studio\data\components\downloads` 的 26 个暂存文件（35,409.4MB）；
- 安装总大小 **59.31 GB → 24.73 GB**（回收 34.58GB；F: 可用空间 186.2 → 220.8GB）；
- 删除旧版重复 `AMS-Studio-Uninstall.exe`，注册表 `UninstallString` 改为 `uninstall.exe`；
- 运行数据完好：kernel/runtime/gateway/voices 均在，无需重装。

## 5. 发布

| 项 | 值 |
|---|---|
| 版本 | **v1.0.1**（launcher/installer 1.0.1；gateway/plugin 仍 1.0.0；kernel 0.2.1-rc.3） |
| GitHub release | v1.0.1，id 429165374，prerelease=false → **latest** |
| feed | **2053**（13 组件；gateway/plugin 复用 v1.0.0 资产；setup/gateway/plugin 版本标注一致） |
| setup | `AMS-Studio-Setup-1.0.1.exe`，3,129,927 B，sha256 `67C3FFFC31F3540A7DE5458402EF4250CF57DE55233449D76D82559608A576E8` |
| launcher | 6,910,464 B，sha256 `B5E54AD5804241AF7411D9642DD96AB13089243313438CA6149EA51B8A9D36A5`（内嵌前端资源） |
| feed sha256 | `78601EB63FCC350AF0D71468B4B5FB7F4DA1E3DECA86B8083577DC525D9D178D` |
| 本地物料 | `09_发布/当前/H5.1_1.0.1_feed2053/` |
| 旧版本标记 | v1.0.0 → previous stable；v0.2.1 → older stable；rc.1/rc.2 仍 superseded |

## 6. 对用户的影响与操作建议

1. **当前安装（v1.0.0）**：已手动清理完成（空间已回收、重复卸载器已删除）；launcher 仍是 1.0.0。
   - 建议：运行 v1.0.1 setup 覆盖安装（启动器升级到 1.0.1，未来安装/更新不再产生 downloads 冗余，也不会再出现双卸载器）。
   - 覆盖安装不会动 `data/` 用户数据。
2. **以后全新安装**：v1.0.1 起，`data/components/downloads` 在安装完成后应为空（正常时 0 字节）；如异常残留，可运行 `ams-launcher.exe --cleanup-downloads`。
3. 安装包不再保留是设计行为，不是缺陷：组件解包后即可删除；如需重装/回滚，重新走 feed 下载即可。

## 7. 仍未解决（不属本次范围）

- GitHub release-assets 大文件下载不稳定（kernel 388MB 会中断，launcher 无断点续传）——见 `NEXT_STEPS.md` P0-1，下载稳定性问题仍在。
- Authenticode 代码签名（P0-2）、agent-music 开箱即用（P0-3）、旧数据迁移文档（P0-4）仍待做。
