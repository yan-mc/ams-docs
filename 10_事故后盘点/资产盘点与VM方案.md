# 事故后资产盘点与隔离开发方案

> 盘点时间：2026-10-20 21:1x（Asia/Shanghai）｜盘点人：Lead
> 范围：`F:/Deepseek_harness/AI音乐处理` 项目、`C:/Users/28988/.dsh` 对话与会话、GitHub `yan-mc` 下 7 个仓库、C 盘恢复副本。
> 原则：只读盘点，未改任何项目/源目录/C 盘文件；本报告是新建文件。
> 本次盘点已做一项零风险保全：把启动器 0.2.0 未提交改动导出为 patch、复制公开更新公钥，并生成 SHA256SUMS，存放于 `10_事故后盘点/backup/`。

## 0. 一句话结论

项目源码和文档基本都在 F 盘，**M0、M1 已完成，M2 打包/暂存完成，M3（组件下载安装）在 2026-10-19 23:50 事故中中断**；GitHub 7 个仓库全部同步，但**仓库当前是公开的**，且**启动器 0.2.0 的未提交改动、128MB 曼波资产包、`ams-release` 更新 feed 尚未进 Git/GitHub**。对话历史恢复了事故前快照中的 49 个 F 工作区会话目录，**22:50–23:52 约 1 小时的对话增量丢失**（含那条清理命令的原文，基本不可恢复）。C 盘软件数据大部分已从 22:50 卷影副本恢复（剪映/豆包等），但 **AMS Studio 的已安装副本（23:45 之后安装）不在快照中，已丢失**；当前开发工具链（Node/pnpm/cargo/uv/ffmpeg）也缺失，需要重装后才能继续构建。

## 1. 时间线与我当前理解

| 时间（10/19） | 事件 | 状态 |
|---|---|---|
| 18:24–22:49 | 现状分析、方案设计、M0 完成、M1 大部分 | 已恢复（快照 + 文件） |
| 22:50:01 | C: 卷影副本 SC2（删除前最后快照） | 用于恢复 |
| 22:50–23:52 | M1 收尾、M2 打包、M3 组件安装开发；23:50:45–23:52:41 批量删除 | **会话正文丢失**，靠文件/日志重建 |
| 23:45 左右 | 启动器 0.1.5 安装包/暂存/GitHub release 完成 | 产物在 F 盘与 GitHub |
| 23:51:27 | M3 `cargo build` 因 `.cargo` 被删，rustc ICE | 0.2.0 未构建成功 |
| 00:07 之后 | 重启、恢复调查、恢复数据 | 已完成大部分 |

事故调查的完整证据、USN、VSS、删除范围见 `F:/Deepseek_harness/C盘事故调查/00_汇总/总报告.md`；这里不重复取证过程。

## 2. 对话历史盘点

### 2.1 当前 `.dsh` 状态

| 项 | 数量/大小 | 说明 |
|---|---|---|
| `C:/Users/28988/.dsh/sessions` | 126 个会话目录 / 80 文件 / 122.4MB | 当前恢复后的会话树 |
| `--F-Deepseek_harness--` | 49 个目录 / 51 文件 / 82.4MB | 本项目工作区；其中 44 个来自 22:50 快照，另 5 个是恢复后的新会话 |
| 主会话 `session-028894f7…` | 4.86MB（当前） | 就是本对话；快照版 4.72MB，当前文件已包含恢复后的新轮次 |
| `workspace.json` 登记 | F 工作区 12 个会话 / E 工作区 1 / MaidMod 4 | **只登记了部分恢复会话**；侧栏可能只显示 12 个 F 工作区会话 |

### 2.2 已恢复的可读副本

| 产物 | 数量/大小 | 位置 |
|---|---|---|
| SC2 快照全量副本 | 83,683 文件 / 14,740.4MB | `C盘事故调查/03_恢复操作/recovered/from_SC2_2026-10-19_2250/` |
| 可读会话导出 | 66 个 MD + 172 个 JSONL + 2 JSON | `C盘事故调查/03_恢复操作/recovered/dsh_sessions_decoded/` |
| 主会话明文 | `rec-session-028894f7…jsonl` 16.43MB；digest 434KB（4,019 行 / 1.93M 字符） | `C盘事故调查/01_项目取证/_evidence/` |
| 事故调查材料 | 93,123 文件 / 17,659.8MB | `F:/Deepseek_harness/C盘事故调查/` |

### 2.3 丢失/不可恢复

- **22:50:01–23:52:41 的会话增量**：M1 收尾、M2 讨论、M3 开发、以及那条批量删除命令的原文；`.dsh/sessions` 在 23:51:02 被同一次删除删掉，快照止于 22:49:57，SSD TRIM 后内容级恢复希望低。
- **主会话在 22:49:57 之后到恢复前的正文**：当前文件是“快照版 + 恢复后新轮次”，中间约 1 小时没有正文。
- **侧栏可见性**：49 个 F 工作区会话目录里只有 12 个登记在 `workspace.json`；其余虽在磁盘上，DSH 侧栏可能看不到。

### 2.4 建议

1. **不要再覆盖 `.dsh`**：先把 `C:/Users/28988/.dsh` 整体复制到 F 盘备份（只读复制，不删原目录）。
2. 需要侧栏显示更多旧会话时，做一个“从会话目录补登记 workspace.json”的小工具；先备份 workspace.json，再逐条导入。
3. 需要人工阅读时，直接用 `dsh_sessions_decoded/_index.md` + 66 个 MD；主会话看 digest 或 16.43MB JSONL。

## 3. 本地项目文件盘点

### 3.1 顶层目录

| 目录 | 文件数 | 内容 |
|---|---|---|
| 00_总览 | 4 | 总报告/术语/资产清单/实操指南 |
| 01_ai歌曲_RVC | 46 | RVC 整合包/音源/念张师分析 |
| 02_ai配音_GPT-SoVITS | 380 | GPT-SoVITS 分析与验证 |
| 03_心月套件 | 2,322 | 心月逆向、解包、证据 |
| 04_pymss_project | 1,359 | pymss 9 仓库研究与原始数据 |
| 05_新方案探讨 | 14,360 | 可行性、心月格式、SVS、开源栈、DSH 架构（含克隆仓库） |
| 06_方案设计 | 19 | 总体设计 + 16 份分册 + 安装布局 |
| 07_执行计划 | 222 | M0/M1/M2 计划与进度、验证、日志、隔离 DSH_HOME |
| 08_项目代码 | 196,318 | 7 个代码仓库 + 构建产物/依赖 |
| 09_发布 | 9 | 0.1.5 release/staging + 曼波 voicepack |
| 99_素材索引 | 5 | 基线、源目录校验 |
| 根 | 2 | README、临时文件.txt（用户要求保留） |

> 全目录合计 **215,052 文件 / 22.86GB**（含依赖、构建产物、克隆仓库、恢复副本）。`08_项目代码` 文件数大是因为 `node_modules`、`.venv`、`target`、PyInstaller build；源码本身只有约几百个文件。

### 3.2 项目阶段

| 阶段 | 状态 | 证据 |
|---|---|---|
| M0（环境/仓库/DSH fork/MSVC/Gateway/插件闭环） | ✅ 完成 | `07_执行计划/M0_进度.md`、各仓库 commit |
| M1（启动器 0.1.5、插件 0.3.0、真实 Runner、端到端） | ✅ 完成 | `M1_进度.md`、`验证/M1_端到端验证.md`、37 tests |
| M2（验签、voicepack、stage-release、文件选择器、0.1.5 发布） | 🟡 大部分完成 | `M2_进度.md`、`09_发布/staging`、GitHub release v0.1.5 |
| M3（组件下载/安装、0.2.0） | 🔴 中断 | 启动器工作区未提交 0.2.0；`launcher_build_m3.log` rustc ICE；无 0.2.0 产物 |

### 3.3 代码仓库本地状态

| 仓库 | 本地 HEAD | 远端 main | 工作区 |
|---|---|---|---|
| ams-launcher | `8e58f8e` | `8e58f8e` | **有未提交改动**：0.2.0 组件下载/安装、`keys/updates-public.pem` 未跟踪；无 0.2.0 构建 |
| ams-harness | `fc326c0b` | `fc326c0b` | 干净；上游 CI 最新 run 失败（fork 的上游 workflow，不是 AMS 构建） |
| ams-plugins | `68b01f9` | `68b01f9` | 干净；5 页签/任务操作/21-21 验证 |
| ams-services | `a90740a` | `a90740a` | 仅 `gateway/.gitignore` 未提交（构建目录忽略项） |
| ams-tools | `30a3cb0` | `30a3cb0` | 干净；msvc2pth v0.4.0，3/3 |
| ams-release | `6d51dc1` | `6d51dc1` | 干净；voicepack/stage 脚本、7 组件清单示例 |
| ams-docs | `49c4c08` | `49c4c08` | 干净；与主文档关键文件哈希一致 |

### 3.4 发布产物

| 产物 | 大小 | 位置 | 状态 |
|---|---|---|---|
| 启动器 0.1.5 NSIS | 1,629,971 B | `09_发布/staging`、`release`、GitHub release | ✅ |
| 启动器 0.1.5 MSI | 2,375,680 B | 同上 | ✅ |
| 启动器 0.1.5 exe | 4,914,176 B | 同上 | ✅ |
| 曼波 voicepack | 134,778,205 B | `09_发布/staging/assets/voicepack-manbo.zip` | ⚠️ 仅本地，GitHub release 未上传 |
| stage-manifest | 902 B | `09_发布/staging/stage-manifest.json` | ✅ 0.1.5，4 个组件 sha256 |
| 0.2.0 安装包 | 无 | — | ❌ M3 构建失败 |

## 4. GitHub 资产盘点

### 4.1 仓库

API 实测：7 个仓库均存在，`private=false`（**公开仓库**），默认分支 main，远端 HEAD 与本地全部一致。

| 仓库 | 大小 | 最新提交 | 说明 |
|---|---|---|---|
| yan-mc/ams-launcher | 272KB | `8e58f8e` | Tauri 启动器；GitHub release v0.1.5（exe/setup/msi） |
| yan-mc/ams-harness | 246,289KB | `fc326c0b` | DSH fork；含完整上游历史，体积大 |
| yan-mc/ams-plugins | 63KB | `68b01f9` | 插件 workspace，0.3.0/21-21 |
| yan-mc/ams-services | 103KB | `a90740a` | Gateway + 真实 Runner + 实验，37 tests |
| yan-mc/ams-tools | 38KB | `30a3cb0` | msvc2pth v0.4.0 |
| yan-mc/ams-release | 15KB | `6d51dc1` | 签名/打包/暂存脚本与示例 |
| yan-mc/ams-docs | 6,073KB | `49c4c08` | 设计文档快照 |

### 4.2 Releases / CI

- 只有 `ams-launcher` 有 release：**v0.1.5**，3 个资产（ams-launcher.exe / setup.exe / msi），下载量 0/1/0。
- `ams-release` **没有 GitHub release**；启动器 0.2.0 代码里指向的 `https://github.com/yan-mc/ams-release/releases/latest/download/ams-components.json` 目前不存在 → 组件安装按钮会 404。
- CI（盘点时最新一次）：services/plugins/launcher/tools/release 均 success；`ams-harness` failure（上游 fork 的 workflow，不阻塞 AMS 代码）；`ams-docs` 无 workflow。

### 4.3 GitHub 侧缺口

1. **仓库是公开的**：设计文档、事故分析、部分本地路径都会公开；如果本意是私有，需要改成 private。
2. **0.2.0 未提交**：组件安装/下载代码只在本地工作区，GitHub 上没有。
3. **更新 feed 不存在**：`ams-release` 无 release，无 `ams-components.json`；voicepack 也没有发布资产。
4. **v0.1.5 release 不含 voicepack**：只有 3 个安装资产，默认音源现在只能从本机 `F:/ai歌曲/0 曼波` 复制。

## 5. C 盘恢复与当前环境

### 5.1 已恢复

- 剪映 `Local/JianyingPro`：28,299 文件 / 7,756MB（快照恢复）；豆包 `Local/Doubao`：3,542 / 131.6MB；
- DSH `.dsh`：126 会话目录 / 122.4MB；附件/缓存等已重建或恢复；
- `.codex`：7,012 文件 / 621.9MB（快照是 8,598 / 1,129.55MB，**部分恢复**）；
- Python 3.12、Git、VS BuildTools + MSVC + Win SDK 10.0.26100、WebView2 151 都还在；
- Rust toolchain 文件还在（`stable-x86_64-pc-windows-msvc`，cargo.exe/rustc.exe 在 toolchain/bin 下），但 `.cargo/bin` 代理和 registry 被删。

### 5.2 仍缺失/受损

| 项 | 状态 | 影响 |
|---|---|---|
| Node.js / npm / pnpm | ❌ 缺失 | 前端/插件/内核构建不能跑；DSH 自带 runtime 有 node/pnpm 可临时借用 |
| cargo/rustc（PATH 代理） | ❌ 缺失 | 可直接调 toolchain/bin；crates 缓存需重下 |
| uv / ffmpeg / gh | ❌ 缺失 | Python 环境/转码/CLI 便利受影响 |
| `.gradle` / `.vscode` / `Local/JetBrains` | ❌ 未恢复 | IDE 缓存/本地历史；项目源码在 F 盘，不影响代码 |
| `Local/Microsoft/WinGet/Packages` | ❌ 未恢复 | 之前用 winget 装的 Node/FFmpeg/uv/ripgrep 包丢失 |
| `Local/AMS Studio` / `Programs/AMS` | ❌ 不存在 | 0.1.5 安装发生在 22:50 快照之后，无法从快照恢复；安装包在 F/GitHub |
| `.codex` | ⚠️ 部分（621.9/1,129.6MB） | Codex 历史部分丢失 |
| 注册表使用痕迹 | ❌ 不可恢复 | 不影响项目 |

> 结论：**代码与文档在 F 盘，没丢；丢的是开发工具链和 C 盘应用安装态**。恢复工具链比恢复应用缓存更重要。

## 6. 缺失/风险清单（按紧急度）

| 级别 | 项 | 说明 | 建议 |
|---|---|---|---|
| P0 | 0.2.0 未提交工作区 | 6 个修改 + 未跟踪公钥；再出一次事故就丢 | 先做 patch/备份分支，再提交；公钥可提交 |
| P0 | 工具链缺失 | 不能构建/测试 | 经用户同意后重装 Node/pnpm/Rust/uv/ffmpeg |
| P0 | GitHub 仓库公开 | 设计/事故资料公开 | 确认是否改为 private |
| P0 | 更新 feed 不存在 | 0.2.0 组件安装不可用 | 建 ams-release release + 上传 signed ams-components.json |
| P1 | voicepack 未发布 | 默认曼波只能本机复制 | 上传到 release 资产或单独离线包，保留 sha256 |
| P1 | 侧栏只登记 12/49 会话 | 旧会话可能看不到 | 备份 workspace.json 后补登记 |
| P1 | 无 0.2.0 构建 | M3 未验收 | 恢复工具链后重构/补测试 |
| P2 | `.codex` 部分丢失 | 非项目数据 | 如需要，从快照再恢复/接受损失 |
| P2 | 无 VM 隔离 | 再次误操作风险 | 见第 7 节 |

## 7. 虚拟机方案

### 7.1 本机条件

- 系统：Windows 10 Pro（Build 26100，实际为 Win11 24H2 内核）
- CPU：i9-13900HX，24C/32T；虚拟化固件开启、SLAT 支持；内存 32GB；
- 磁盘：C 221GB 空闲 / D 398GB / E 559GB / F 365GB；
- GPU：RTX 4080 Laptop 12GB（**不能直通给普通 VM**）；
- 现状：**Hyper-V 未启用**（`HypervisorPresent=False`、`vmms` 不存在）；未装 VMware/VirtualBox；Windows Sandbox 未启用；WSL 存在但仅适合 Linux。

### 7.2 结论

**强烈建议用一个 Windows 虚拟机专门跑“第二个 DSH / fork 内核 / 插件 / 启动器开发与危险操作测试”**，但 **GPU 推理（RVC/pymss）仍留在宿主机**。普通消费级 NVIDIA 显卡（尤其笔记本）不支持 Hyper-V/VMware/VirtualBox 的 GPU 直通；不要在这上面浪费时间。

### 7.3 方案对比

| 方案 | 成本 | 优点 | 缺点 | 适用 |
|---|---|---|---|---|
| **Hyper-V（推荐）** | Windows Pro 内置；需启用功能+重启；需 Windows ISO | 系统自带、快照/检查点、性能好、无第三方 | 需要 Windows 专业版；GPU 不可用；首次配置略麻烦 | 持久隔离开发 VM |
| VMware Workstation Pro | 免费个人版，需 Broadcom 账号下载 | 快照/共享文件夹/3D 较成熟 | 无 GPU 直通；需下载/账号 | Hyper-V 不方便时替代 |
| VirtualBox | 免费，约 100MB | 简单、跨平台 | 性能/3D 较弱 | 轻量测试 |
| Windows Sandbox | Win Pro 内置 | 秒级、一次性、最适合安装/卸载测试 | 不持久、每次重置、不能长期开发 | 安装器/卸载器冒烟 |
| WSL2 | 免费 | Linux 工具链好 | 不能跑 Windows DSH/Tauri；无 Windows GUI | 仅 Linux 服务/脚本 |
| 双系统/第二台机器 | 需分区/硬件 | 有完整 GPU | 切换麻烦/成本高 | 需要 GPU 全隔离时 |

### 7.4 推荐配置

- 宿主：保持 RTX 4080 做 RVC/pymss GPU；不要让 VM 抢 GPU；
- VM：Windows 11 Pro x64，8 vCPU、16GB 内存、120GB 动态 VHDX（放 D: 或 E:）、4GB 显存（WDDM 基本显示）；
- 网络：NAT 或 Internal Switch；不要桥接宿主机内网；
- 隔离：不共享 C 盘；如需传文件，用只读共享或复制测试夹具；VM 内使用**独立 GitHub 账号/token**（最小权限）和独立 DeepSeek Key；
- 快照：装好基础环境后打 baseline 快照；每次危险测试前再打快照；
- VM 内装：Node 24 + pnpm、Rust stable + VS Build Tools C++、WebView2、Python 3.12、DSH fork；约 25–40GB；
- GPU 相关：RVC/pymss 继续在宿主机跑，VM 里只做 UI/插件/Gateway mock/启动器构建。

### 7.5 需要下载/授权的软件

| 项 | 大小/方式 | 是否必须 |
|---|---|---|
| Windows 11 x64 ISO（官方） | 约 6–7GB，微软官网/媒体创建工具 | 必须（Hyper-V/VMware 都需要 guest OS） |
| Hyper-V 功能启用 | 系统内置，管理员 + 重启 | 推荐方案，无下载 |
| VMware Workstation Pro | 免费个人版，Broadcom 账号下载（数百 MB） | 可选替代 |
| VirtualBox | 约 100MB | 可选轻量替代 |
| Windows Sandbox 功能 | 系统内置，启用+重启 | 可选（安装器冒烟） |
| VM 内开发工具：Node LTS、pnpm、Rust stable、VS Build Tools C++、WebView2 | 合计约 5–10GB 下载 | 必须（VM 内构建） |

> 需要你确认：是否允许我下载 Windows 11 ISO / 启用 Hyper-V / 下载 VMware 或 VirtualBox；以及 VM 放在 D 还是 E。**未确认前我不下载、不启用系统功能。**

## 8. 接下来建议的顺序

### 第 1 步：先保资产（今天，低风险）

1. 把启动器未提交的 0.2.0 改动导出为 patch（`git diff` + 未跟踪的 public.pem），复制到 `09_发布/backup` 或 `10_事故后盘点/backup`；【本次已完成：`10_事故后盘点/backup/launcher-0.2.0-wip.patch`、`updates-public.pem`、`SHA256SUMS.txt`】
2. 把 `C:/Users/28988/.dsh` 整体只读复制到 F 盘（不要再覆盖）；
3. 把 `09_发布/staging`（尤其 128MB voicepack）复制一份到备份目录；
4. 确认 GitHub 仓库公开/私有策略；如需私有再改（不改代码）。

### 第 2 步：恢复开发环境（需授权）

1. 重装 Node LTS + pnpm（或先借用 DSH 自带 runtime）；
2. 恢复 Rust PATH 代理/组件，清理并重下 crates；
3. 重装 uv、ffmpeg；
4. 用 0.1.5 的构建链验证：`pnpm build` + `cargo test` + `tauri build`；
5. 再构建 0.2.0，补 M3 测试（组件清单/下载/安装/sha256/失败回滚）。

### 第 3 步：继续项目（M2 收尾 → M3 → M4）

- 创建 `ams-release` GitHub release，上传签名后的 `ams-components.json`（以及可选组件包）；
- 把 voicepack 作为 release 资产/离线包发布；
- 0.2.0 启动器接真实 GitHub feed，完成下载/校验/安装/回滚；
- 再考虑内核/Gateway 整包进安装器、签名证书、训练/TTS 等。

### 第 4 步：VM 隔离

- 先启用 Hyper-V + 安装 Windows guest（如你同意）；
- 在 VM 内复现 0.2.0 构建/安装/卸载测试；宿主机只做 GPU 后端；
- 以后所有“第二个 DSH”、插件热更、清理类脚本都在 VM 里跑。

## 9. 证据索引

- 事故调查总报告：`F:/Deepseek_harness/C盘事故调查/00_汇总/总报告.md`
- 对话历史抢救：`C盘事故调查/01_项目取证/对话历史抢救.md`
- 精确被删清单：`C盘事故调查/02_C盘取证/快照对比_精确被删清单.md`
- 恢复副本：`C盘事故调查/03_恢复操作/recovered/from_SC2_2026-10-19_2250/`
- 可读会话：`C盘事故调查/03_恢复操作/recovered/dsh_sessions_decoded/_index.md`
- 项目进度：`07_执行计划/M0_进度.md`、`M1_进度.md`、`M2_进度.md`
- M3 失败日志：`07_执行计划/launcher_build_m3.log`
- 发布暂存：`09_发布/staging/stage-manifest.json`

## 10. 需要你拍板

1. GitHub 7 仓库现在是**公开**的，是否改回私有？（建议私有，或至少 ams-docs/ams-harness 私有）
2. 是否同意下载 Windows 11 ISO、启用 Hyper-V / 安装 VMware/VirtualBox？VM 放 D 还是 E？
3. 是否同意现在恢复 Node/pnpm/Rust/uv/ffmpeg 工具链（需要下载）？
4. 未提交的 0.2.0 改动，是否允许我提交到一个 `m3-wip` 分支（先不合并）？
