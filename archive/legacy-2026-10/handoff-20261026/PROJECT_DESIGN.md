# AMS Studio 项目整体设计书

> 当前状态见根目录 [`CURRENT_STATE.md`](CURRENT_STATE.md)：版本/feed/哈希/门禁以它为准。

版本：接手版  
更新时间：2026-10-26  
适用对象：后续接手的工程/AI/验收人员

## 1. 核心理念

AMS Studio 是一个本地优先的 AI 音乐工作站：

- 音频、音源、数据集、模型、项目全部保存在本机；
- GPU 任务本地执行，不依赖云端音频服务；
- Gateway 统一管理任务、GPU 锁、进度、产物；
- DSH 作为 Agent/Kernel，提供页面、工具和可扩展 Skill；
- 发布通过签名 feed + 组件包，而不是更新整个安装目录；
- 任何模型/音源/训练数据都可审计、可回滚。

一句话：

本地数据 + 本地 GPU + Gateway 任务化 + DSH Agent 化 + 签名组件更新。

## 2. 产品目标

### 2.1 用户目标

- 导入歌曲/音频/数据集；
- RVC 音源炼制与换声；
- pymss 人声分离；
- TTS 音源合成；
- TTS 数据集与音源炼制；
- 混音/母带/评测；
- 工程管理、任务管理、音源管理；
- 通过 Agent 用自然语言操作上述能力。

### 2.2 工程目标

- Windows 本地可安装；
- 数据只在 F:/AMS-Studio/data；
- 组件可独立升级；
- 发布可签名、可校验、可回滚；
- Agent 工具与 UI 插件解耦；
- 测试数据、日志、临时文件不进入正式发布。

## 3. 总体架构

Launcher  
-> Kernel/DSH Host  
-> AMS Host/Client Plugins  
-> Gateway  
-> Runners/Runtimes  
-> data

发布侧：

setup + feed + kernel + gateway + plugin + runtime + manifest + signature

### 3.1 Launcher

Tauri 2 桌面程序。

职责：

- 安装/卸载；
- 识别安装根和 data；
- 检查组件；
- 下载 feed 组件；
- 启动/停止 Kernel、Gateway；
- 显示版本/状态；
- 提供修复/清理入口。

关键约束：

- 所有 AMS 数据在安装目录 data；
- 不写 C 盘；
- 组件下载可校验 SHA；
- setup 必须同时生成 uninstall.exe 别名，launcher 才能识别安装状态。

### 3.2 Kernel / DSH

DSH 是 Agent/Host 运行时。

职责：

- 加载 profile；
- 加载 Host/Client 插件；
- 提供 Skill、工具、Agent 会话；
- 托管 Web UI；
- 通过 Web 认证启动欢迎页。

关键修复：

- Kernel 启动时 host launch URL 的 cookie 必须注入欢迎页；
- 否则 desktop welcome: Web authentication failed，Kernel 打不开。

### 3.3 Host/Client 插件

Host 插件：

- 注册工具；
- 提供 /ams/api 路由；
- 管理 session/token；
- 代理 Gateway 任务/SSE。

Client 插件：

- AMS 页面；
- 12 个页签；
- 表单、任务列表、播放、音源/数据集界面。

当前 music-bundle = plugin-core + plugin-ui。

### 3.4 Gateway

FastAPI 本地服务。

职责：

- 任务提交/排队/取消/状态；
- GPU/CPU worker 和锁；
- SSE 事件；
- 产物/报告；
- runner 注册；
- ports.json/gateway.token；
- 任务持久化。

任务类型：

- ping
- mock.long
- convert.msvc
- convert.rvc
- separate.pymss
- train.rvc
- tts.synthesize
- tts.dataset.prepare
- train.tts
- dataset.prepare
- evaluate.audio
- mix.master

### 3.5 Runtimes

- runtime.rvc：RVC 训练/转换；
- runtime.pymss：人声分离；
- runtime.tts：Piper 推理；
- runtime.tts-train：Piper/VITS 训练；
- tool.msvc：MSVC 容器转换；
- tool.vcrun：VC++ 运行库；
- runtime.pymss.python：pymss 便携 Python。

runtime 是数据目录的一部分，不进入 Git。

### 3.6 Agent 插件

独立仓库：ams-agent-music。

目标：

- 给 Agent 一个音乐处理模式；
- 不依赖 AMS UI 插件；
- 所有操作通过 ams_* 工具；
- 统一路径/确认/审计安全；
- 内置本地 ASR。

能力：

- mode/status/inventory；
- dataset inspect/audit/clean；
- TTS text clean；
- dataset prepare；
- RVC/TTS training；
- job submit/status/wait/cancel；
- TTS synthesize；
- local ASR；
- evaluate audio；
- release preflight。

当前版本：0.2.0，20 tools，独立 bundle，GitHub 已发布。

## 4. 数据与格式

### 4.1 数据根

F:/AMS-Studio/data

子目录：

- datasets
- voices
- tts_voices
- models
- projects
- artifacts
- runtime
- components
- kernel
- config
- logs
- backups
- trash

### 4.2 RVC 音源

voices/<name>/

- <name>.pth
- <name>.index
- voice.json

### 4.3 TTS 音源

tts_voices/<name>/

- <name>.onnx
- <name>.onnx.json
- voice.json

### 4.4 TTS 数据集

datasets/tts/<name>/

- metadata.csv，格式 id|text
- wav/<id>.wav

可选：

- metadata.cleaned.csv（不覆盖原文件）

### 4.5 工程

projects/<id>/project.json

包含：

- assets
- steps
- outputs
- 时间戳
- 任务记录

### 4.6 任务

Gateway job：

- id
- type
- status
- progress
- stage
- params
- project_id
- error
- engine
- gpu_lock
- created/started/finished

### 4.7 发布 feed

ams-update/1：

- schema
- channel
- sequence
- components
- setup
- signature
- product_version

签名：Ed25519，签名覆盖除 signature 外的完整清单。

### 4.8 Manifest

ams.release/1：

- sequence
- product_version
- setup
- components
- commits
- generated_at

## 5. UI 页签

当前实际 14 页签（v1.0.0，`plugin-ui` 1.0.0）：

1. 项目
2. 实时变声
3. 音色库
4. TTS 批量
5. 音源炼制
6. 数据集工作室
7. 评测器
8. 混音母带
9. 人声分离
10. 歌曲换源
11. MSVC 工具
12. TTS 语音合成
13. TTS 数据集
14. TTS 音源炼制

> H4 及更早文档写的"12 页签"是 H2 时代口径；H3 起新增 TTS 数据集、TTS 音源炼制，实际 14 个。

## 6. 发布体系

### 6.1 阶段

- H0：基线与冻结
- H1：核心工程/回归
- H2：核心音频链路与正式 0.2.1
- H3：1.0.0-rc.1 RC
- H4：RC 全量验收
- H5：v1.0.0 正式发布

### 6.2 当前发布状态（2026-10-26 17:05 起）

**v1.0.0（latest，已发布）**：

- v1.0.0，GitHub release id 424030311，prerelease=false
- feed **2052**（13 组件），签名有效
- setup `AMS-Studio-Setup-1.0.0.exe`（3,126,682 B / `8275319B…`）
- launcher / gateway / plugin 均 **1.0.0**；kernel **0.2.1-rc.3**（独立版本线）
- runtimes：rvc/pymss/tts/tts-train 0.1.0（Hugging Face）

**历史版本（资产全部保留）**：

- v0.2.1：previous stable / feed 2049（回滚源）
- v1.0.0-rc.2：superseded / feed 2051（H4 候选）
- v1.0.0-rc.1：superseded / feed 2050（含已知 dev-mode launcher 缺陷）
- v0.2.0：archived

H4：**SIGNED（2026-10-26 17:05）**；H5：**已发布（latest=v1.0.0）**。详见 `CURRENT_STATE.md`。

## 7. 已完成

> 以 `CURRENT_STATE.md` §4 为准：**H0–H5 全部完成，v1.0.0 已发布并 latest**。

- Launcher/安装器/卸载保留 data；干净安装/升级/回滚实测通过；
- DSH Kernel（含 Web 认证热修；
- Gateway 任务系统（12 类任务，含 mix.master 修复）；
- RVC 训练/转换（3/5/12 分钟、无 index）；
- pymss 分离（4 stems）；
- TTS 合成（中英）与 TTS 数据集/训练（质量封存）；
- MSVC 转换；
- 项目/任务/SSE；
- **UI 14 页签 CDP（14/14，三套环境）**；
- **GPU E2E（基础 6/6、12 分钟长音频、训练链 dataset→train→注册→convert）**；
- 签名 feed/组件发布（feed 2052，latest 切换）；
- 独立 Agent 插件（20 tools + skill，手工安装可用）；
- 构建可复现（env.ps1 + pnpm tauri build + 内嵌资源断言）；
- 文档/发布物料归一（CURRENT_STATE、09_发布 SOP、_接管审计 04–12）。

## 8. 未完成（v1.0.0 之后）

> 详细清单、验收标准与优先级见根目录 **[`NEXT_STEPS.md`](NEXT_STEPS.md)**。摘要：

### P0（影响普通用户顺利安装/使用）

1. **GitHub release-assets 大文件下载不稳**（kernel 388MB 会中断；launcher 无续传）→ 需要 HF 镜像或分片续传/离线包；
2. **安装包/launcher 未 Authenticode 签名**（SmartScreen 拦截）；
3. **agent-music 不能"下载即用"**：审计日志硬编码 `F:/AMS-Studio/...`、ASR 依赖路径与 runtime 实际布局不一致、vosk 模型不在 feed、未纳入 feed/launcher 一键安装；
4. **旧数据迁移文档缺失**：真实音源/数据集/模型仍需从 `F:/_trash/ams-clean-20261026_1445` 恢复，需写清导入步骤。

### P1

5. 生产私钥管理与是否轮换（当前仍在仓库工作树，已仓库外备份）；
6. 伞仓 `AMS-Studio` submodule 指针陈旧（0.2.0 时代）；
7. 严格远端 feed 全新安装在真实网络下的复验；
8. 面向 v1.0.0 的安装/升级/回滚/离线包用户说明书（旧说明书是 0.2.0 口径）；
9. VC 真实麦克风+RVC 30–60 分钟长稳；
10. agent-music 20 tools 自动化回归；
11. v1.0.0 launcher "安装音乐插件到内核" 的端到端 UI 自动化。

### P2

12. TTS 听感调优封存、ASR 小模型精度、中文歌声数据集、MSVC 样本、混音非完整 DAW、评测基准库、Kernel 版本串统一、构建工具链便携化。

## 9. 封存项

- TTS 训练质量调优：封存；不继续调参。
- TTS 音源上传：封存；GitHub/HF 不放 TTS 音源。
- VM GPU 验收：不采用；使用独立物理机或本机干净目录。
- 本地曼波/闲云/闲云TTS：保留在隔离区，不进入 GitHub 发布资产。
- 旧 harness 副本/旧 runtime 副本：可回滚归档，不作为源码。

## 10. 已知问题与修复

### 10.1 Kernel 打不开

错误：

desktop welcome: Web authentication failed

原因：

- host launch URL 返回 303 + set-cookie；
- 欢迎页第二次 fetch 没带 cookie；
- kernel 启动中止。

修复：

- hostCookie 注入 session/请求；
- H2/H3 kernel rc.3 已包含；
- 备份 app.asar.bak-auth-20261026114220。

### 10.2 两个卸载文件

- AMS-Studio-Uninstall.exe：NSIS 真正卸载器；
- uninstall.exe：launcher 识别安装目录的别名；
- 不是重复安装。

### 10.3 Gateway C 盘 token

- 不是 Gateway bug；
- 是 PowerShell $home 自动变量覆盖 --ams-home；
- 使用 $amsHome 后 token/ports 正确落在 AMS_HOME；
- 已清理 C:/Users/28988/runtime。

### 10.4 干净 H3 启动退出

原因：

- 复制 kernel 不完整，resources/app.asar 缺失；
- 复制了生产 electron-data/lockfile；
- setup 缺 uninstall.exe 别名。

修复：

- 新 setup 生成别名；
- 补全 resources；
- 删除干净 electron-data；
- 使用正确 AMS_HOME 启动 Gateway；
- H3B clean 实测：Gateway 8618、token/ports 在 H3B、20 tools/skill、selftest ok。

## 10.5 rc.1 launcher dev-mode 构建（已修，v1.0.0）

- rc.1 setup 内 launcher 是裸 `cargo build --release`（未启用 tauri custom-protocol），二进制无内嵌前端资源，干净环境会依赖 `localhost:1420`、存在白屏风险；
- H4 曾只验证"launcher 可打开"，未验证渲染；
- 修复：rc.2 起用 `pnpm tauri build --no-bundle` + build-installer.ps1 硬断言"必须内嵌前端资源"；v1.0.0 已重建。

## 10.6 Gateway mix.master runner 丢失（已修）

- TTS 提交（ams-services 5fd1a4a）重写 runner 注册表时漏掉 `mix.master`，`/health` job_types 缺混音类型；
- 修复：ams-services 2f00523（恢复注册 + 按 type 取描述），gateway pytest 41 passed；rc.2 起组件已含。

## 10.7 agent-music 0.2.0 现状（未修，见 NEXT_STEPS P0-3）

- 可从 GitHub release 下载 zip 并手工安装，20 tools + skill 能加载（实测 selftest ok）；
- 但审计日志硬编码 `F:/AMS-Studio/data/logs/agent-music-load.log`；
- ASR 依赖路径与实际 runtime 布局不一致（实际 `runtime/tts-train/0.1.0/runtime.tts-train-0.1.0/python`），vosk 模型不在 feed；
- 未纳入 feed/launcher 一键安装。

## 10.8 GitHub release-assets 大文件不稳（已知限制）

- kernel 388MB 单连接多次中断（84MB / 251MB）；
- 当前用本地镜像/离线包绕过；长期方案见 NEXT_STEPS P0-1。

## 11. 安全边界

- 只写 F:/AMS-Studio/data；
- 禁止 C 盘 AMS 数据；
- 不直接递归删除；
- 删除/移动先隔离；
- 训练/发布/上传需确认；
- token 不离开 Host；
- 音频/转写默认不外传；
- TTS 音源不进入公开 release。

## 12. 接手顺序（v1.0.0 发布后）

1. 阅读 [`README.md`](README.md)（项目入口）与 [`CURRENT_STATE.md`](CURRENT_STATE.md)（唯一事实来源：版本/feed/哈希/门禁）；
2. 阅读 [`NEXT_STEPS.md`](NEXT_STEPS.md)（当前未完成事项与优先级）；
3. 阅读本文（PROJECT_DESIGN.md）与 PROJECT_FILE_MAP.md，确认架构与目录职责；
4. 阅读 `_接管审计/04`–`12`（发布整理、环境恢复、构建复现、GitHub 执行记录、H3/H4 验收、H4 签字、H5 发布）；
5. 安装：用 GitHub release v1.0.0 的 setup（或 `09_发布/当前/H5_1.0.0_feed2052/AMS-Studio-Setup-1.0.0.exe`）安装到新目录，启动 launcher 下载 feed 组件；
6. 数据恢复：按 `_接管审计/13_F盘清理与验证环境说明.md` 从 `F:/_trash/ams-clean-20261026_1445` 恢复音源/工程/模型；
7. 下一步工作按 NEXT_STEPS P0/P1 顺序推进（下载稳定性 → agent-music 修复 → 用户说明书 → 质量项）。

## 13. 不能删/不能动

- C:/Users/28988/.dsh
- C:/Users/28988/AppData/Local/JianyingPro
- F:/Deepseek_harness/AI音乐处理/08_项目代码/_harness_full
- ams-harness Junction 指向的 _harness_full
- 隔离区里的曼波/闲云/闲云TTS/Vosk/xianyun640/tts 数据集，除非用户明确批准

## 14. 当前清理状态（2026-10-26 17:25 清理后）

- **F:/AMS-Studio 已彻底卸载并删除（含 data）**；注册表 `AMS Studio` / `AMS Studio CleanTest` / `HKCU\Software\yanmc` 均已清理；用户可全新安装 v1.0.0。
- **F 盘测试环境已清理**：`AMS-H4-H3`、`AMS-H4-RC2`、`AMS-H5-VERIFY`、`AMS-H2-VERIFY`、`AMS-BUILD-VERIFY`、`_gw-selftest*`、`_edge-cdp-*`、`_logs`、`_agent_music_test`、`_trash` 内测试残留均已删除（明细见 `_接管审计/13_F盘清理与验证环境说明.md`）。
- **保留**：`F:/_trash/ams-clean-20261026_1445`（4.4GB 用户数据：曼波/闲云/xianyun_zh、datasets、projects、tts_voices、vosk 模型、日志）；`F:/AMS-keys-backup-20261026/`（签名密钥备份）。
- 项目工作区约 9GB；所有 GitHub 源码仓库 clean 并已推送；文档/发布物料已归一。
- GitHub release 资产零删除（v0.2.0 archived、rc.1/rc.2 superseded、v0.2.1 previous stable、v1.0.0 latest）。

## 15. 结论

AMS Studio 已完成 **H0–H5 全流程**：架构、发布链、音频链路、Agent 插件均已成形；**v1.0.0 已发布并切换 latest**。

当前项目状态不是"功能未完成"，而是"**发布后工程化收尾**"：

- P0：下载稳定性（大文件/续传/镜像）、代码签名、agent-music 开箱即用、旧数据迁移文档；
- P1：密钥管理、伞仓指针、真实网络复验、用户说明书、VC 长稳、自动化回归；
- P2：质量与长期债（TTS 听感、ASR 精度、中文歌声、MSVC 样本、评测基准）。

详见 [`NEXT_STEPS.md`](NEXT_STEPS.md) 与 [`CURRENT_STATE.md`](CURRENT_STATE.md)。
