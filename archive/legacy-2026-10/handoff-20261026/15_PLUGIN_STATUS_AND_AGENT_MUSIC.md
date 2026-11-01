# 15. 音乐插件"未安装"判定问题 + agent-music 现状（2026-10-30）

范围：`F:\AMS-Studio`（用户 v1.0.1 安装）→ 源码修复 → v1.0.2 发布（feed 2054）  
相关提交：`ams-launcher` **85f17f2**（launcher/setup 1.0.2）

---

## 1. 问题一：音乐插件显示"未安装"

### 1.1 现象

用户安装完全部组件、并执行了"安装音乐插件到内核"之后，launcher 的「组件状态」卡片里"音乐插件"仍然显示 **未安装**。

### 1.2 实际状态：插件其实已经安装成功

现场证据（用户 v1.0.1 安装）：

| 证据 | 内容 |
|---|---|
| 组件包 | `data/components/plugin.ams.music.bundle/`：`ams-music-bundle-1.0.0.tgz`、`ams-plugin-core-1.0.0.tgz`、`ams-plugin-ui-1.0.0.tgz` 均在 |
| Kernel profile | `data/kernel/dsh/profiles/desktop/node_modules/@ams/`：`music-bundle`、`plugin-core`、`plugin-ui` 三个模块都在 |
| profile package.json | `dsh.profile.bundles` 含 `@ams/music-bundle`；依赖为三个 `file:` tgz |
| 安装操作日志 | `profiles/desktop/.plugin-manager/logs/` 下 2026-10-30 23:39:32–34 有 3 次 plugin add 操作（pnpm 安装成功） |

即插件本体、profile 安装都正常，只是 launcher 的状态显示错误。

### 1.3 根因

`ams-launcher/src-tauri/src/lib.rs` 的 `component_status()`（旧代码）：

```rust
let plugins_dir = ams_home().join("components").join("plugin.ams.music.bundle");
let plugin_pkgs = plugins_dir.join("ams-music-bundle-0.3.0.tgz").is_file()
    || plugins_dir.join("ams-plugin-core-0.3.0.tgz").is_file();
```

**硬编码了 `0.3.0` 的 tgz 文件名**。v1.0.x 的组件包是 `*-1.0.0.tgz`，所以：
- 组件检查永远 false → 状态永远是"未安装"；
- 这与 rc.1 时期"安装音乐插件到内核"按钮写死 `0.3.0` 是同一类版本硬编码 bug（按钮在 v1.0.1 已修，但状态判定漏了）。

### 1.4 修复（v1.0.2）

1. 动态发现：按前缀扫描 `ams-music-bundle-*` / `ams-plugin-core-*` / `ams-plugin-ui-*` 的 `.tgz`，不再写死版本号；
2. 区分三态：
   - `未安装`：组件包和 profile 都没有；
   - `已下载（未安装到内核）`：组件包在，但 profile `node_modules/@ams` 不完整；
   - `已安装到内核`：三个 `@ams/*` 模块都在（绿色 ok）；
3. 新增诊断命令 `ams-launcher.exe --component-status`，把同样判定输出到
   `data/logs/launcher-component-status.json`。

### 1.5 验证

用 v1.0.2 setup 装到干净目录 `F:\_ams-plugin-test\app`，逐步构造状态：

| 场景 | 结果 |
|---|---|
| A 无插件组件 | `未安装`，ok=false |
| B 放入 3 个 1.0.0 tgz | `已下载（未安装到内核）`，ok=false |
| C 再创建 profile `@ams/{music-bundle,plugin-core,plugin-ui}` | `已安装到内核`，ok=true |
| D **用户真实安装**（`AMS_HOME=F:/AMS-Studio/data`） | **`已安装到内核`，ok=true** ✅ |

## 2. 问题二：agent-music（"方便 DSH 操作一切"的插件）现状

### 2.1 身份

- 仓库：`yan-mc/ams-agent-music`，包名 `@ams/agent-music`（Host 插件，**独立于** `@ams/music-bundle`）；
- 能力：**20 个 `ams_*` 工具** + 1 个 Skill（`ams-music-processing`）+ 本地 Vosk ASR；
- 工具覆盖：模式进入/状态/清单、数据集检查/审计/清理、TTS 文本清洗、RVC/TTS 数据集准备、音色删除到回收站、任务提交/状态/列表/取消/等待、TTS 合成、ASR 转写、音频评测、文档查询、发布预检；目的是让 DSH/Kernel 里的 Agent 能操作整个 AMS；
- 版本：**v0.2.0**（repo HEAD `0408006`，release v0.2.0 资产 2 个：zip 21,813 B + SHA）。

### 2.2 当前状态（"未完成"具体指什么）

| 项 | 状态 |
|---|---|
| 代码/构建 | ✅ 完成：dist zip、release v0.2.0 可下载，SHA 一致 |
| 手工安装可用 | ✅ 已验证：按 README 复制 `@ams/agent-music` + `@ams/agent-music-bundle` 到 Kernel profile、加入 profile 依赖与 `dsh.profile.bundles`、重启 Kernel → `tools-registered count=20`、`skill-list ams-music-processing`、`selftest ok` |
| 是否随 AMS 安装 | ❌ **不在 feed**（v1.0.0–v1.0.2 的 13 个组件里没有它） |
| 是否可一键安装 | ❌ launcher 没有对应按钮/命令，只能手工改 profile |
| ASR（`ams_transcribe`） | ❌ 默认不可用：依赖路径与实际 runtime 布局不一致（实际是 `runtime/tts-train/0.1.0/runtime.tts-train-0.1.0/python` 与 `.../deps`），且 `models/vosk-cn` 与 `tools/asr` 不在 feed |
| 审计日志路径 | ⚠️ 硬编码 `F:/AMS-Studio/data/logs/agent-music-load.log`（自定义安装目录会写错位置或丢日志） |
| engines 声明 | ⚠️ `engines.dsh: 0.2.1-rc.2`，Kernel 实际 0.2.1-rc.3（实测未阻塞加载） |
| 当前用户安装中 | ❌ **未安装**：profile 只有 music-bundle/plugin-core/plugin-ui，`dsh.profile.bundles` 无 `@ams/agent-music-bundle` |
| 自动化回归 | ❌ 只有 selftest + 手工证据 |

### 2.3 结论与建议

**结论**：agent-music 是"**代码完成、可手工安装、但未进入正式发布链**"的插件；因此普通用户安装完 AMS 后不会得到它，DSH 也就没有那 20 个操作工具。这就是之前 `NEXT_STEPS.md` 里的 P0-3。

**建议下一步（可作为 v1.1）**：
1. 修 agent-music 0.2.1：审计日志改 `AMS_HOME/logs`、ASR 运行时路径改为"实际目录自动探测 + 环境变量覆盖"、engines 更新；
2. 打包为 feed 组件（kind=`plugin`），或由 launcher 增加"安装 Agent 插件到内核"一键按钮（复用已修好的动态 tgz 安装逻辑）；
3. ASR 依赖（vosk 57MB + 中文模型 65MB）作为可选组件/可选下载，不强制进主安装；
4. 把 20 tools 的 smoke 脚本纳入 QA（P1-6）。

> 注：`F:/_trash/ams-clean-20261026_1445` 里保留了 `models/vosk-cn` 与 `tools/asr`（用户旧数据），需要时可手工恢复给 ASR 用。
