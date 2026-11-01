# AMS Studio — 本地优先 AI 音乐工作站

本地优先的 Windows AI 音乐工作站：**Tauri launcher + DSH Kernel + Cordis 插件 + FastAPI Gateway + 本地 RVC / pymss / TTS / TTS-train / MSVC runtime**。音频、音源、数据集、模型与工程全部留在本机，GPU 任务本地执行；发布通过 **签名 feed + 组件包** 更新，不重装整个安装目录。

> **唯一事实来源：[`CURRENT_STATE.md`](CURRENT_STATE.md)**。版本号、feed、哈希、发布状态、门禁结论一律以它为准；其他任何文档中的数字与之冲突时，视为历史快照（已作废）。
> 根目录现在只保留 4 个入口文件：`README.md`（本文）、`CURRENT_STATE.md`、`PROJECT_DESIGN.md`、`PROJECT_FILE_MAP.md`；其余历史文档已归档到 `11_`、`12_`、`13_`。

---

## 当前状态（2026-10-31 15:45）

| 项 | 状态 |
|---|---|
| 稳定版（GitHub latest） | **v1.1.1「嘉音Studio」**，feed **2056**，setup `0BBF5C86…D924`（快捷方式/图标 + 壁纸 + 音乐制作模式） |
| Previous stable（回滚源） | v1.1.0（feed 2055）、v1.0.2（feed 2054）、v1.0.1（feed 2053）、v1.0.0（feed 2052）、v0.2.1（feed 2049） |
| Superseded | v1.0.0-rc.1（feed 2050）、v1.0.0-rc.2（feed 2051）、v0.2.0（archived） |
| H4 验收 | **PASS / SIGNED**（2026-10-26 17:05） |
| H5–v1.1 发布 | **v1.0.0 → v1.0.1 → v1.0.2 → v1.1.0 → v1.1.1（latest，嘉音Studio）**（`_接管审计/12`、`14`、`15`、`16`、`17`） |
| 当前验证环境 | **无长期环境**（本轮 app3 干净安装验证后清理） |
| 全新安装包 | GitHub release `v1.1.1`，或本地 `09_发布/当前/V1.1_1.1.1_feed2056/AMS-Studio-Setup-1.1.1.exe` |
| 用户当前安装 | `F:/AMS-Studio`：**launcher v1.1.1 + 组件托管插件**（Agent 0.2.2，20 tools/壁纸/音乐制作模式）；桌面快捷方式已就绪 |
| 真实数据唯一副本 | `F:/_trash/ams-clean-20261026_1445/AMS-Studio/data`（4.39GB，勿删） |
| 源码基线 | `08_项目代码/` 下 8 个仓库均 main、clean；launcher/installer **1.1.1（嘉音Studio）**，gateway/plugin 1.0.0，kernel 0.2.1-rc.3，agent-music **0.2.2（壁纸 + 音乐制作模式，已进 feed）** |

阶段门禁速览：H0 PASS → H1 PASS → H2 PASS → H3 PASS（UI 14 页签 + GPU E2E）→ **H4 SIGNED** → **H5 v1.0.0 发布** → **H5.1 v1.0.1** → **H5.2 v1.0.2（latest）**。后续未完成事项见 [`NEXT_STEPS.md`](NEXT_STEPS.md)。

---

## 目录导航

| 路径 | 作用 | 入口 |
|---|---|---|
| [`00_总览/`](00_总览/) | 项目总览、早期入口 | 目录内 README |
| [`01_ai歌曲_RVC/`](01_ai歌曲_RVC/) | RVC 歌曲/音源素材与实验记录 | — |
| [`02_ai配音_GPT-SoVITS/`](02_ai配音_GPT-SoVITS/) | 配音/TTS 旧实验 | — |
| [`03_心月套件/`](03_心月套件/) | 心月套件工程资料 | — |
| [`04_pymss_project/`](04_pymss_project/) | pymss 人声分离工程资料 | — |
| [`05_新方案探讨/`](05_新方案探讨/) | DSH/架构调研、repo-sparse 证据 | — |
| [`06_方案设计/`](06_方案设计/) | 总体设计、产品约定 | — |
| [`07_执行计划/`](07_执行计划/) | 早期阶段计划、验收计划 | — |
| [`08_项目代码/`](08_项目代码/) | 全部可执行源码与 Git 工作副本（8 仓库 + `_harness_full`） | `PROJECT_FILE_MAP.md` §3 |
| [`09_发布/`](09_发布/) | 发布 feed、setup、manifest、当前 release 物料（已重组：`当前/` + `历史/`） | `09_发布/README.md`（发布 SOP） |
| [`10_事故后盘点/`](10_事故后盘点/) | 事故记录、盘点 | — |
| [`11_验收与发布记录/`](11_验收与发布记录/) | **历史**：H0–H4 验收/发布/缺陷记录 | [`11_验收与发布记录/README.md`](11_验收与发布记录/README.md) |
| [`12_专项记录/`](12_专项记录/) | **历史**：Agent 音乐插件、TTS 专题记录 | [`12_专项记录/README.md`](12_专项记录/README.md) |
| [`13_历史计划/`](13_历史计划/) | **历史**：旧计划、B 方案、安装说明书 | [`13_历史计划/README.md`](13_历史计划/README.md) |
| [`99_素材索引/`](99_素材索引/) | 素材索引 | — |
| [`_接管审计/`](_接管审计/) | 接管方案 00、接手审计 01–03、工作流报告 04–08、H3/H4/H5 验收签字 09–12、F 盘清理 13、运行/构建脚本 `ops/`、日志 `logs/` | `_接管审计/00_接管方案_20261026.md` 起 |
| [`_工具/`](_工具/) | 本机构建/签名辅助工具（Rust toolchain 等） | — |
| [`_归档_20261025/`](_归档_20261025/) | 阶段日志、测试备份、旧阶段归档 | — |
| [`AMS-Studio/`](AMS-Studio/) | 顶层伞仓（submodule 仍 pin 在 0.2.0 时代，**不可作交付基线**） | — |
| `F:/AMS-Studio/` | 用户当前安装：launcher 1.1.1 + 组件托管插件（Agent 0.2.2），桌面/开始菜单快捷方式已创建 | — |
| `F:/_trash/ams-clean-20261026_1445/` | 隔离区：真实数据唯一副本（4.39GB，勿删） | — |
| `F:/AMS-keys-backup-20261026/` | 生产签名私钥仓库外备份 | — |

---

## 接手 / 开发者入口（按顺序读）

1. [`CURRENT_STATE.md`](CURRENT_STATE.md) — **唯一事实来源**：版本矩阵、GitHub 发布现状、门禁、已知问题。
2. [`NEXT_STEPS.md`](NEXT_STEPS.md) — **v1.0.0 之后的未完成事项**：P0/P1/P2、agent-music 现状、验收标准。
3. [`PROJECT_DESIGN.md`](PROJECT_DESIGN.md) — 项目整体设计：架构、数据/格式、UI 页签、发布体系、已知问题与修复、安全边界。
4. [`PROJECT_FILE_MAP.md`](PROJECT_FILE_MAP.md) — 文件结构与职责地图（目录、源码仓库、数据目录、隔离区）。
5. [`_接管审计/`](_接管审计/) — 接管方案 00、审计 01–03、工作流 04–08、H3/H4/H5 验收签字 09–12、F 盘清理 13。
6. [`09_发布/README.md`](09_发布/README.md) — 发布 SOP、版本矩阵、资产 ↔ GitHub 映射（只从 `09_发布/当前/` 取物料）。
7. 历史证据索引：[`11_验收与发布记录/README.md`](11_验收与发布记录/README.md)、[`12_专项记录/README.md`](12_专项记录/README.md)、[`13_历史计划/README.md`](13_历史计划/README.md)。

---

## 常用路径 / 命令速查

| 用途 | 路径 / 约定 |
|---|---|
| 工作区根 | `F:/Deepseek_harness/AI音乐处理` |
| 源码 | `08_项目代码/`（launcher / harness(kernel) / plugins / services / tools / release / docs / agent-music） |
| 当前发布物料 | `09_发布/当前/`（H2/H3/H4/H5 历史 + `H5.1_1.0.1_feed2053/`、`H5.2_1.0.2_feed2054/`、**`V1.1_1.1.0_feed2055/`**） |
| 发布 SOP / 资产映射 | `09_发布/README.md` |
| 全新安装包 | `09_发布/当前/H5.1_1.0.1_feed2053/AMS-Studio-Setup-1.0.1.exe`（或 GitHub release v1.0.1） |
| 测试安装环境 | **已清理**（原 `F:/AMS-H4-H3` 等已删除，见 `_接管审计/13_F盘清理与验证环境说明.md`） |
| 旧安装 | **F:/AMS-Studio 已卸载删除（含 data）** |
| 真实数据（唯一副本） | `F:/_trash/ams-clean-20261026_1445/AMS-Studio/data`（4.39GB，勿删） |
| 环境装配脚本 | `_接管审计/ops/env.ps1`（装配 DSH node/pnpm、`.build-tools`、Rust toolchain 的 PATH） |
| runtime 下载脚本 | `_接管审计/ops/download_runtimes.py` |
| 接管审计日志 | `_接管审计/logs/` |

**路径变量约定（重要）**：Gateway / launcher 启动前设置 `AMS_HOME` 时，PowerShell 中**不要**用 `$home` / `$HOME` 作变量名——它是只读自动变量，会覆盖 `--ams-home` 参数，导致 token/ports 写到 C 盘用户目录。统一用 `$amsHome` / `$safeTarget` / `$projRoot`：

```powershell
$ErrorActionPreference = 'Stop'
$projRoot = 'F:\Deepseek_harness\AI音乐处理'
$amsHome = 'F:\AMS-Studio\data'   # 全新安装后的默认数据目录
# Gateway 以 --ams-home $amsHome 启动；具体参数与发布前检查以 09_发布/README.md 为准
```

Windows 下所有路径操作使用单引号常量 + `-LiteralPath`；移动/清理前先 `Resolve-Path` 确认绝对路径。

---

## 安全边界（接手后必须遵守）

- **不写 C 盘 AMS 数据**：不触碰 `C:/Users/28988`、`C:/Program Files`、`C:/Windows`、`C:/ProgramData`；token/ports 必须落在 `AMS_HOME`。
- **不递归删除**：删除/清理前先确认路径，优先移动到隔离区（`F:/_trash`），不直接 `Remove-Item -Recurse`。
- **用户资产不入库**：用户音色、模型权重、真实工程、训练数据不进入源码仓库；GitHub release 只放可公开的代码与组件资产，TTS 音源不进入公开 release。
- **token 不离开 Host**；音频与转写默认不外传。
- **生产签名私钥**：`ams-release/examples/keys/private.pem`（仓库外备份 `F:/AMS-keys-backup-20261026/`），不得外传或写入发布物。
- **SDK/Junction 保护**：`08_项目代码/_harness_full` 与 `ams-harness` Junction 不可删除。
- **发布物料来源单一**：只从 `09_发布/当前/` 取；`09_发布/历史/` 只作回滚与审计，不参与发布。

---

## 文档索引

| 位置 | 内容 |
|---|---|
| [`CURRENT_STATE.md`](CURRENT_STATE.md) | 唯一事实来源（含"明确作废的旧数字"表，引用旧文档前先查） |
| [`NEXT_STEPS.md`](NEXT_STEPS.md) | **v1.0.0 之后的未完成事项（P0/P1/P2）与推荐顺序** |
| [`PROJECT_DESIGN.md`](PROJECT_DESIGN.md) | 项目整体设计书 |
| [`PROJECT_FILE_MAP.md`](PROJECT_FILE_MAP.md) | 文件结构与职责地图 |
| [`11_验收与发布记录/README.md`](11_验收与发布记录/README.md) | H0–H4 验收、发布、缺陷、资产清单（历史证据） |
| [`12_专项记录/README.md`](12_专项记录/README.md) | Agent 音乐插件、TTS 音源炼制（历史证据） |
| [`13_历史计划/README.md`](13_历史计划/README.md) | 旧计划、B 方案、安装说明书（历史快照） |
| [`_接管审计/`](_接管审计/) | 审计 01–03、发布整理 04、环境恢复 05、构建复现 06、GitHub 执行 07、文档归一 08、H3/H4/H5 验收签字 09–12、F 盘清理 13、安装器修复 14、插件状态 15、嘉音Studio v1.1 改版 16 |

> 归档规则：被移动的历史 `.md` 文首都加有"历史快照（已归档）"横幅（`H3_ASSET_INVENTORY.csv`、`BASELINE_MANIFEST.json`、`TEST_DATA_MANIFEST.csv` 为数据文件，未加横幅但同样是历史快照）。迁移明细（旧路径→新路径→字节数→sha256）见 [`_接管审计/08_文档迁移清单.md`](_接管审计/08_文档迁移清单.md)。

---

## 维护规则

1. 任何版本 / feed / 哈希 / 发布状态变化，**先改 `CURRENT_STATE.md`**，再改其他文档；其他文档不得另立数字。
2. 根目录不再新增散落文档：新的历史证据放入 `11_` / `12_` / `13_` 对应目录，并在该目录 `README.md` 索引登记。
3. 归档文档只移动 / 追加说明，不删除。
4. 发布物料整理、GitHub release 标题/正文/资产变更必须记录在 `09_发布/README.md`。


