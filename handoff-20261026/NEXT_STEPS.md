# AMS Studio 后续任务清单

更新：2026-10-30 23:05（Asia/Shanghai）  
状态基准：[`CURRENT_STATE.md`](CURRENT_STATE.md)（版本/feed/哈希/发布状态以它为准）  
适用范围：**v1.0.1（latest）之后仍未完成的工作**。

---

## 0. 已完成（供对照）

- **H0–H5 全部完成**：H4 已签字，v1.0.0 已发布（feed 2052）。
- **H5.1 v1.0.1 已发布（feed 2053，latest）**：修复安装/更新磁盘占用与双卸载器（见 §1.1）。
- H3 验收证据补齐：UI 14 页签 CDP（14/14，两套环境）、GPU E2E（基础 6/6、12 分钟长音频、训练链 dataset→train→注册→convert）。
- 发布物料/文档/仓库归一（`CURRENT_STATE.md`、`09_发布/README.md`、`_接管审计/04–14`）。
- 已修复（v1.0.0 之前/期间）：Gateway mix.master 注册丢失、launcher dev-mode 构建（白屏）、构建不可复现、插件 tgz 硬编码、launcher UA 硬编码。

---

## 1.1 2026-10-30 已修复（v1.0.1）

| 问题 | 根因 | 修复 |
|---|---|---|
| 安装后磁盘占用异常（全量安装约 35.4GB 冗余） | launcher `download_parts_to` 合并分卷后不删分卷；`install_component_blocking` 解包后不删 zip；无残留清扫 | 分卷合并即删、解包即删、安装循环收尾清扫、启动自愈；新增 `--cleanup-downloads`；v1.0.1 已发布 |
| 安装目录出现两个卸载器 | NSIS `WriteUninstaller AMS-Studio-Uninstall.exe` + `CopyFiles uninstall.exe`（launcher 只认 uninstall.exe） | setup 只写 `uninstall.exe`；注册表 UninstallString 指向它；升级自动删旧 `AMS-Studio-Uninstall.exe` |

证据：`_接管审计/14_下载缓存与安装器重复问题_20261030.md`；当前 v1.0.0 安装已手动回收 34.58GB（59.31→24.73GB）并删除重复卸载器。

---

## 1. 未完成事项（按优先级）

### P0 — 影响"普通用户能顺利安装/使用"

| # | 事项 | 现状/证据 | 建议方案 | 验收标准 |
|---|---|---|---|---|
| P0-1 | **GitHub release-assets 大文件下载不稳** | kernel 388MB 单连接两次中断（84MB / 251MB）；launcher 下载器无 resume/retry | ① kernel/runtime 增加 HuggingFace 镜像并在 feed 增加 `mirrors`；② launcher 下载器实现分片+断点续传；③ 提供离线安装包 | 全新网络/干净目录从 feed 完整安装并启动 Kernel+Gateway，中途可断点续传 |
| P0-2 | **安装包/launcher 未 Authenticode 签名** | setup/launcher 均 NotSigned；SmartScreen 会拦截 | 购买/申请代码签名证书；setup 与 launcher 签名；发布流程加入验签 | Windows 干净机器双击安装无 SmartScreen 阻断（或明确发布说明指引） |
| P0-3 | **agent-music 不能"下载即用"** | 见 §2 专项 | 修硬编码/路径 + 提供安装方式（launcher 一键或 feed 组件） | 普通用户按文档 5 分钟内装上 20 tools+skill；ASR 有明确可选依赖说明 |
| P0-4 | **用户从旧数据迁移的路径未文档化** | 真实数据仍只在 `F:/_trash/ams-clean-20261026_1445`（voices/datasets/projects/tts_voices/models） | 写"数据恢复/导入"说明：把 voices/tts_voices/datasets/projects/models 复制回新安装 `data/` 并校验 | 用户按文档可恢复曼波/闲云/工程/vosk 模型 |

### P1 — 发布链/运维可信度

| # | 事项 | 现状/证据 | 建议方案 |
|---|---|---|---|
| P1-1 | 生产私钥管理 | 生产 Ed25519 私钥仍在仓库工作树 `ams-release/examples/keys/private.pem`（已仓库外备份 `F:/AMS-keys-backup-20261026/`） | 决策：迁出仓库 + CI secret；是否轮换（需重签内置新公钥的 launcher 并公告） |
| P1-2 | 伞仓 submodule 指针陈旧 | `AMS-Studio/` 的 7 个 submodule 仍 pin 在 0.2.0 时代 | 更新 submodule 指针并提交 tag；或明确声明"以 08_项目代码 main 为交付基线" |
| P1-3 | 严格远端 feed 全新安装只到 PARTIAL | 网络原因，已用本地镜像链路验证；镜像 feed 已随测试环境清理 | 与 P0-1 一起解决后重跑并留证 |
| P1-4 | 面向用户的安装/升级/回滚说明书 | 现有 `13_历史计划/AMS-Studio-完整安装与使用说明书.md` 是 0.2.0/feed 2042 旧版 | 按 v1.0.0 重写：安装、组件下载、离线包、升级、回滚（装 v0.2.1）、卸载保留 data、安全边界 |
| P1-5 | VC 真实链路长稳 | H2 只做了 device enumerate + passthrough 30 分钟；真实麦克风+RVC 长稳未验 | 30–60 分钟真实音频压测、设备热插拔、恢复 |
| P1-6 | agent-music 20 tools 自动化回归 | 只有 selftest + 手工证据（PATH_DENIED/CONFIRMATION_REQUIRED） | 增加脚本化 smoke：mode/status/inventory/dataset_audit/tts_text_clean/evaluate/release_check |
| P1-7 | 安装后首次启动体验 | launcher "安装音乐插件到内核" 在 v1.0.0 已修 tgz 动态发现，但未做 UI 自动化验证 | 干净安装 → 启动 Kernel → 装插件 → 重启 → 14 页签的端到端自动化（可沿用 qa/ui-cdp.ps1） |

### P2 — 质量/长期债

| # | 事项 | 说明 |
|---|---|---|
| P2-1 | TTS 训练质量 | 音素 pad 已修、链路可用；听感/断句调优按用户要求封存 |
| P2-2 | ASR 精度 | Vosk small 模型，用于路由/校验，不是最终字幕；可用更大模型替换（需要下载器/许可证说明） |
| P2-3 | 中文歌声数据集与听感验收 | 缺高质量中文歌声数据集，训练听感未验 |
| P2-4 | MSVC 正向样本有限 | 仅有少量 .msvc 样本；建议补样本集 |
| P2-5 | 混音/母带范围 | 轻量 DSP，不是完整 DAW（发布说明已声明） |
| P2-6 | 评测基准库 | 指标可复现但无基准库，无法判断"好坏" |
| P2-7 | Kernel 内部版本串 | 内部版本仍 0.2.1-rc.2 而 feed 写 0.2.1-rc.3（H4 接受例外）；下个 kernel 版本统一 |
| P2-8 | 旧文档/资产 | v0.2.0 release 仍保留（archived）；如需彻底清理需用户确认（当前零删除策略） |
| P2-9 | 构建工具链外部依赖 | env.ps1 依赖 D 盘 DSH node/pnpm 与 C 盘 Tauri makensis 缓存；建议在 `_工具` 固化便携副本 |

---

## 2. agent-music 专项现状（2026-10-26 实测）

**结论：可以正常下载、可以安装生效，但不是"下载即用"，且 ASR 依赖不完整。**

- 仓库/版本：`yan-mc/ams-agent-music` v0.2.0（private 仓库；release 资产 `ams-agent-music-0.2.0.zip` 21,813 B，sha256 `6117C69E…D14A`；本地与远端一致）。
- **实测安装（干净 AMS v1.0.0 环境）**：
  - 从 GitHub release 下载 zip → 解压 → 按 README `INSTALL.md` 把 `@ams/agent-music` 与 `@ams/agent-music-bundle` 复制到 Kernel profile `node_modules/@ams/`，在 profile `package.json` 添加依赖与 `dsh.profile.bundles` → 重启 Kernel。
  - 结果：**插件加载成功**：`tools-registered count=20`、`skill-list ams-music-processing`、`skill-content bytes=3182`、`selftest ok=true`（Gateway 1.0.0）。
- **问题（需要在 0.2.1 修）**：
  1. **审计日志路径硬编码** `F:/AMS-Studio/data/logs/agent-music-load.log`（lib/index.js），装到别的目录/别的盘时日志会写错位置或丢失；应改为 `AMS_HOME/logs/`。
  2. **ASR 依赖路径文档与实际 runtime 布局不一致**：README 写 `runtime/tts-train/0.1.0/python/python.exe`、`.../deps`，实际发布 runtime 解包后是 `runtime/tts-train/0.1.0/runtime.tts-train-0.1.0/python`（多一层）；`models/vosk-cn/vosk-model-small-cn-0.22` 也不在 feed 中。
  3. 因此 `ams_transcribe` 在干净安装默认不可用（需要用户自备 Vosk 依赖+模型，或从 `F:/_trash/ams-clean-20261026_1445/AMS-Studio/data/{models,tools}` 恢复）；其余 19 个工具可用。
  4. `engines.dsh` 声明 `0.2.1-rc.2`，实际 Kernel `0.2.1-rc.3`（实测未阻塞加载，但声明应更新）。
  5. 未纳入 feed / launcher 一键安装，只能手工 profile 安装。
- **建议**：0.2.1 版本修上述 1/2/4；发布说明写清 ASR 可选依赖；评估把 bundle+plugin 纳入 feed（或 launcher 增加"安装 Agent 插件"按钮）。

---

## 3. 推荐执行顺序

1. **v1.0.1（运维修复，1–2 天）**：P0-1 下载续传/镜像、P0-3 agent-music 修复、P1-4 用户安装说明书、P0-4 数据迁移文档；重新验证干净安装。
2. **v1.0.x 质量线**：P1-5 VC 长稳、P1-6 agent-tools 回归、P2-1/2 质量项评估。
3. **v1.1（可选）**：agent-music 入 feed、Authenticode、私钥轮换/CI 签名、中文歌声数据集。

## 4. 相关文档

- 版本/发布真值：[`CURRENT_STATE.md`](CURRENT_STATE.md)
- 发布 SOP：[`09_发布/README.md`](09_发布/README.md)
- 验收证据：`_接管审计/09_H3_验收报告.md`、`10_H4_验收报告.md`、`11_H4_签字.md`、`12_H5_发布报告.md`
- F 盘清理与验证环境说明：`_接管审计/13_F盘清理与验证环境说明.md`
- 历史计划（已归档，不再作为依据）：`13_历史计划/`
