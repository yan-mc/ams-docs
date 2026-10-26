# 13. F 盘清理、卸载与验证环境说明（2026-10-26）

执行：接手负责人（Lead）  
目的：清理本轮产生的测试目录/临时产物，彻底卸载 `F:/AMS-Studio`（含 data），交付一个"干净机器 + 保留用户数据备份"的状态，供用户亲自全新安装 v1.0.0。  
操作日志：`_接管审计/logs/cleanup-fdrive-20261026.log`

---

## 1. 清理前校验（先证明可回滚/无用户数据丢失）

| 项 | 结果 |
|---|---|
| 用户数据唯一副本 `F:/_trash/ams-clean-20261026_1445/AMS-Studio/data` | **完好**：4.39GB / 关键目录：voices 8 文件 1,299.2MB（曼波/闲云/xianyun_zh）、datasets 2,063 文件 1,268.9MB、projects 24 文件 716.1MB、tts_voices 7 文件 181.1MB（闲云/huayan/lessac）、assets 4/187.5MB、tools 4,258/324.6MB（含 ASR 依赖）、models 15/107MB（含 Vosk 中文模型）、config/logs/qa/exports |
| 三套测试环境中的音色/数据集/工程 | 均为上述备份的**副本**，非唯一数据；runtime 可从 HF 重新下载 |
| 在跑进程 | CDP Edge（F 盘独立 profile）、测试 Gateway、测试 Kernel 已定位并停止 |

## 2. 已删除内容（仅本轮测试产物；逐条校验路径）

| 路径 | 内容 | 说明 |
|---|---|---|
| `F:/AMS-H4-H3` | 离线恢复验证环境（~66.88GB / 138,668 文件） | 仅用于 rc.1 环境恢复与 H3 证据 |
| `F:/AMS-H4-RC2` | rc.2 干净安装验证环境（~59.24GB） | H4 干净安装/UI 验证 |
| `F:/AMS-H5-VERIFY` | v1.0.0 干净安装验证环境（~59.24GB） | H5 发布后干净安装/UI 验证 |
| `F:/AMS-H2-VERIFY` | H2 安装/升级/回滚测试残留（data 标记 1 文件） | 已无程序文件 |
| `F:/AMS-BUILD-VERIFY` | 构建产物安装测试残留（data 1 文件） | — |
| `F:/_gw-selftest`、`_gw-selftest2`、`_gw-selftest-h5` | Gateway 独立自测 AMS_HOME | 各 6 文件 |
| `F:/_edge-cdp-profile`、`_edge-cdp-cache`、`_edge-cdp-tmp` | UI CDP 专用 Edge profile/cache/temp | 605.1MB / 2,205 文件 等 |
| `F:/_logs` | 构建/发布临时日志（h5-build、build-repro 等） | 20.7MB；关键结论已写报告 |
| `F:/_agent_music_test` | agent-music release 下载/解压实测临时目录 | 18 文件 |
| `F:/_trash/ams-test-residue-20261026` | 失败下载残片（kernel 84MB 等） | 127.9MB |
| `F:/_trash/build-artifacts-20261026` | installer 备份 .bak 等 | 11.9MB |
| `F:/AMS-Studio` | **旧安装彻底卸载 + 目录（含 data）删除** | 先运行 `AMS-Studio-Uninstall.exe /S`（程序文件删除、data 保留、注册表清理），再删除整个目录；data 149 文件/9.54MB |

**清理后 F 盘剩余 AMS 相关目录**：`F:/_trash/ams-clean-20261026_1445`（用户数据）、`F:/AMS-keys-backup-20261026`（签名密钥备份）。  
**F 盘可用空间**：清理前约 113GB → 清理后约 **290GB**（三套环境回收约 185GB）。

## 3. 注册表清理

| 键 | 处理 |
|---|---|
| `HKCU\...\Uninstall\AMS Studio` | 由卸载器删除 |
| `HKCU\...\Uninstall\AMS Studio CleanTest`（0.2.0，指向不存在目录） | 已删除 |
| `HKCU\Software\yanmc\AMS Studio CleanTest` | 已删除；空 `HKCU\Software\yanmc` 一并删除 |

清理后 HKCU 下无任何 AMS Studio 卸载项，可干净全新安装。

## 4. 保留内容（不要删）

1. `F:/_trash/ams-clean-20261026_1445` — 用户真实数据：
   - `data/voices/曼波`、`data/voices/闲云`、`data/voices/xianyun_zh`
   - `data/tts_voices/闲云`、`zh_CN-huayan-medium`、`en_US-lessac-medium`
   - `data/datasets/xianyun640`、`data/datasets/tts`
   - `data/models/vosk-cn`（ASR 模型）、`data/tools/asr`（ASR 依赖）
   - `data/projects`、`data/config`、`data/logs`、`data/qa`、`data/exports`、`data/assets`
2. `F:/AMS-keys-backup-20261026/private.pem`、`public.pem` — 生产签名密钥仓库外备份。
3. `F:/Deepseek_harness/AI音乐处理` — 项目工作台（源码/发布物料/文档/证据）。
4. 项目内发布物料 `09_发布/当前/H5_1.0.0_feed2052/AMS-Studio-Setup-1.0.0.exe` — 可直接安装。

## 5. 用户全新安装指引（v1.0.0）

1. 安装包：GitHub release `v1.0.0` 的 `AMS-Studio-Setup-1.0.0.exe`，或本地
   `F:/Deepseek_harness/AI音乐处理/09_发布/当前/H5_1.0.0_feed2052/AMS-Studio-Setup-1.0.0.exe`
   （3,126,682 B，sha256 `8275319B04A041C1249DD24BF4E9C741F3965F8E60AD1BCA3E2CAD127B64C4FA`）。
2. 双击安装（默认目录 `F:/AMS-Studio`，可 /D 改目录）；数据始终在 `<安装根>/data`，不写 C 盘。
3. 启动 launcher → 检查组件（feed 2052）→ 下载并安装全部组件（约 19GB，GitHub + HuggingFace）。
   - **已知问题**：GitHub release-assets 下载 kernel（388MB）可能中断；若发生，重试或执行
     `ams-launcher.exe --install-components`（已安装的组件会跳过）。
4. 启动 Gateway → 启动 Kernel → 安装音乐插件 → 重启 Kernel → 打开「AMS 音乐」（14 页签）。
5. 恢复个人数据（可选）：把 `F:/_trash/ams-clean-20261026_1445/AMS-Studio/data/` 下的
   `voices/`、`tts_voices/`、`datasets/`、`projects/`、`models/vosk-cn/`、`tools/asr/` 复制到新安装
   `data/` 对应目录（先备份新目录；不要整目录覆盖 config/logs）。
6. agent-music（可选）：从 `yan-mc/ams-agent-music` release v0.2.0 下载 zip，按 `INSTALL.md` 手工
   安装到 Kernel profile；ASR 需额外把 `models/vosk-cn` 与 `tools/asr` 放到上面的位置（详见
   `NEXT_STEPS.md` §2 与本次实测记录）。

## 6. 本次 agent-music 安装实测结论（供参考）

在 v1.0.0 干净环境中按 README 手工安装 `ams-agent-music 0.2.0`（GitHub release zip，sha256 `6117C69E…D14A`）：

- 插件加载成功：`tools-registered count=20`、`skill-list ams-music-processing`、`skill-content bytes=3182`、`selftest ok=true`（Gateway 1.0.0）。
- 已知问题：审计日志硬编码 `F:/AMS-Studio/data/logs/agent-music-load.log`；ASR 依赖路径与实际 runtime 目录层级不一致（实际 `runtime/tts-train/0.1.0/runtime.tts-train-0.1.0/python`），且 `models/vosk-cn` 不在 feed；`engines.dsh` 声明 0.2.1-rc.2 而 Kernel 为 rc.3（未阻塞加载）。
- 结论：**能装上、20 tools/skill 生效；但不是"下载即用"，ASR 需自备依赖/模型**。修复建议见 `NEXT_STEPS.md` §2 / P0-3。

## 7. 验证

- 清理后 `F:/` 根目录无 `AMS-*` 测试环境（仅保留 keys 备份与 `_trash` 用户数据）。
- 注册表无 AMS 卸载项；`C:/AMS-Studio` 不存在（历史已确认）。
- 项目仓库 8 个均 clean 且与 origin 同步（清理仅动 F 盘测试目录，不动源码/发布物料）。
- `09_发布/SHA256SUMS-当前.txt` 已按清理后的 36 个文件重新生成（验证用 local-mirror feed 已移除）。
