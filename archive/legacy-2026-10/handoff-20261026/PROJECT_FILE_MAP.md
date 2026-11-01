# AMS Studio 文件结构与职责地图

更新时间：2026-10-26

## 0. 阅读约定

- 本文只描述结构、职责和入口，不复制实现细节。
- GitHub 源码仓库和本地工作副本是两套东西：
  - GitHub 是交付源；
  - F:/Deepseek_harness/AI音乐处理 是工作台、发布物料、证据和归档。
- **F:/AMS-Studio 已于 2026-10-26 彻底卸载并删除（含 data）**，等待用户全新安装 v1.0.0。
- 旧安装的 101GB 生产数据已整体移到 F:/_trash/ams-clean-20261026_1445（现存约 4.4GB 用户数据）；F 盘测试环境已清理（详见 `_接管审计/13_F盘清理与验证环境说明.md`）。

## 1. 顶层工作区

F:/Deepseek_harness/AI音乐处理

| 路径 | 作用 |
|---|---|
| 00_总览 | 项目总览、入口文档 |
| 01_ai歌曲_RVC | RVC 歌曲/音源相关素材与实验记录 |
| 02_ai配音_GPT-SoVITS | 配音/TTS 相关旧实验 |
| 03_心月套件 | 心月套件工程资料 |
| 04_pymss_project | pymss 分离工程资料 |
| 05_新方案探讨 | DSH/架构调研、repo-sparse 证据 |
| 06_方案设计 | 总体设计、产品约定 |
| 07_执行计划 | 阶段计划、验收计划 |
| 08_项目代码 | 所有可执行源码和 Git 工作副本 |
| 09_发布 | 发布 feed、setup、manifest、当前 release 物料 |
| 10_事故后盘点 | 事故记录、盘点 |
| 11_验收与发布记录 | 历史：H0–H4 验收、发布、缺陷、资产清单（索引见该目录 README.md） |
| 12_专项记录 | 历史：Agent 音乐插件、TTS 音源炼制（索引见该目录 README.md） |
| 13_历史计划 | 历史：旧阶段计划、B 方案、安装说明书（索引见该目录 README.md） |
| _接管审计 | **当前交接/验收/发布证据**：00 接管方案、01–03 审计、04 发布整理、05 环境恢复、06 构建复现、07 GitHub 执行、08 文档迁移、09 H3 验收、10 H4 验收、11 H4 签字、12 H5 发布、13 F 盘清理、qa/ 证据、ops/ 脚本 |
| _工具 | 构建/签名/辅助工具 |
| _归档_20261025 | 历史归档 |
| _trash 相关 | 不在此目录；F:/_trash 是隔离区（保留用户数据） |
| 根级 md | 5 个入口：README.md、CURRENT_STATE.md、NEXT_STEPS.md、PROJECT_DESIGN.md、PROJECT_FILE_MAP.md |

## 2. 根级文档（2026-10-26 文档归一 + v1.0.0 发布后）

根目录只保留 5 个入口文件：

| 文件 | 作用 |
|---|---|
| README.md | 项目唯一入口：介绍、当前状态、目录导航、接手入口、路径速查、安全边界 |
| CURRENT_STATE.md | 唯一事实来源：版本矩阵、GitHub 发布现状、门禁、已知问题、已作废数字 |
| NEXT_STEPS.md | **v1.0.0 之后的未完成事项、优先级、验收标准（P0/P1/P2）** |
| PROJECT_DESIGN.md | 项目整体设计书（含已完成/未完成/已知问题/清理状态） |
| PROJECT_FILE_MAP.md | 本文件 |

历史文档已归档到 `11_`/`12_`/`13_`，按主题查索引：

| 目录 | 内容 |
|---|---|
| 11_验收与发布记录/README.md | H0–H4 验收、发布、缺陷、资产清单（历史证据） |
| 12_专项记录/README.md | Agent 音乐插件、TTS 音源炼制（历史证据） |
| 13_历史计划/README.md | 旧阶段计划、B 方案、安装说明书（历史快照） |
| _接管审计/00_接管方案_20261026.md | 本次接管评估与执行结果（非历史） |
| _接管审计/04–13 | 发布整理、环境恢复、构建复现、H3/H4/H5 验收/签字/发布、F 盘清理 |

历史文档逐条索引见各目录 README；迁移明细见 `_接管审计/08_文档迁移清单.md`。

## 3. 08_项目代码

F:/Deepseek_harness/AI音乐处理/08_项目代码

| 目录 | 职责 |
|---|---|
| _harness_full | DSH/DeepSeek Harness 源码实体 |
| ams-harness | 指向 _harness_full 的 Junction，不是独立副本 |
| ams-launcher | Tauri 2 桌面 launcher、安装器、组件下载/启动 |
| ams-services | Gateway、VC service、实验脚本 |
| ams-plugins | DSH Host/Client 插件 monorepo |
| ams-release | 发布清单、签名工具、schema、QA |
| ams-docs | 项目文档仓库 |
| ams-agent-music | 独立 Agent 音乐处理插件仓库 |
| .build-tools | 构建辅助 |
| AMS-Studio 子模块 | 由顶层 AMS-Studio 仓库组织 |

### 3.1 ams-launcher

| 路径 | 作用 |
|---|---|
| src/ | Vue 前端：安装状态、组件、更新、启动 UI |
| src-tauri/src/lib.rs | Tauri 命令、AMS_HOME、组件下载、Gateway/Kernel 启动 |
| src-tauri/tauri.conf.json | Tauri 打包配置 |
| installer/ams-setup.nsi | NSIS 安装器；当前已生成 uninstall.exe 别名 |
| installer/build-installer.ps1 | 构建 setup |
| installer/stage/ | setup 打包输入（launcher.exe） |
| dist/ | 构建输出 |
| package.json | 前端依赖和脚本 |

### 3.2 ams-services

| 路径 | 作用 |
|---|---|
| gateway/app/ | FastAPI Gateway |
| gateway/app/main.py | CLI、应用工厂、ports/token、uvicorn |
| gateway/app/config.py | AMS_HOME、目录、端口、token 配置 |
| gateway/app/api/ | jobs、health、projects、voices、TTS 等路由 |
| gateway/app/runners/ | RVC、pymss、TTS、训练、评测 runner |
| gateway/tests/ | Gateway 测试 |
| gateway/scripts/ | 构建/辅助脚本 |
| vc-service/ | 实时 VC 服务 |
| experiments/ | 性能/分离实验脚本和结果（cache/pymss 重资产已移走） |

### 3.3 ams-plugins

| 路径 | 作用 |
|---|---|
| packages/plugin-core | Host 插件：/ams/api、session、通用 music_* 工具、Gateway bridge |
| packages/plugin-ui | Client 插件：AMS 页面、**14 页签**、任务/音频 UI |
| （独立）ams-agent-music/client | @ams/agent-music-ui：内核运行时换标 + 壁纸注入 |
| （独立）ams-agent-music/mode | @ams/agent-music-mode：音乐制作模式自动工作目录（data 动态解析） |
| packages/music-bundle | profile bundle：插入 plugin-core + plugin-ui |
| packages/agent-music 或独立目录 | Agent 插件独立仓库，不属于 music-bundle |

### 3.4 ams-release

| 路径 | 作用 |
|---|---|
| tools/manifest.py | ams-update/1 清单校验、Ed25519 签名/验签 |
| tools/build-*.ps1/py | setup、组件、runtime、voicepack 构建 |
| schemas/ams-update-1.schema.json | feed schema |
| examples/ | 示例清单、签名 key |
| qa/ | H3/安装 smoke 脚本 |
| tests/ | manifest 测试 |
| RELEASE_2042.md | 历史发布说明 |

### 3.5 ams-docs

项目文档仓库；内容与本地 H0-H4、安装、缺陷、设计文档同步。

### 3.6 ams-agent-music

独立 Agent 插件仓库：

| 路径 | 作用 |
|---|---|
| src/index.js | Host 插件、20 个 ams_* 工具、Skill provider |
| lib/index.js | 构建产物 |
| asr.py | 本地 Vosk ASR |
| skills/ams-music-processing | 主 Skill + references |
| asr-deps / asr-model | 本地 ASR 依赖和中文模型（gitignored） |
| bundle/ | 独立 @ams/agent-music-bundle |
| dist/ | 正式插件 zip + SHA |
| README/INSTALL/CHANGELOG | 安装与版本说明 |

## 4. 09_发布

2026-10-26 重组、2026-10-30 增补 H5.1 后结构（发布/回滚只认 `当前/`）：

| 路径 | 作用 |
|---|---|
| README.md | 发布 SOP：版本矩阵、资产↔GitHub 映射、发布/回滚步骤、禁止事项 |
| SHA256SUMS-当前.txt | `当前/` 下全部文件 sha256 |
| 当前/H2_0.2.1_feed2049/ | H2 回滚源：feed 2049、setup、kernel、catalog、voicepack、gateway/plugin stageg2、service-vc、manifest、SHA、PROVENANCE |
| 当前/H3_1.0.0-rc.1_feed2050/ | H3 superseded 候选物料 |
| 当前/H4_1.0.0-rc.2_feed2051/ | H4 候选物料（rc.2） |
| 当前/H5_1.0.0_feed2052/ | v1.0.0 发布物料（previous stable）：feed 2052、setup 1.0.0、gateway/plugin 1.0.0、manifest、SHA、release-notes |
| 当前/H5.1_1.0.1_feed2053/ | v1.0.1 补丁物料（previous stable）：feed 2053、setup 1.0.1、manifest、SHA、release-notes |
| 当前/H5.2_1.0.2_feed2054/ | v1.0.2 补丁物料（previous stable） |
| 当前/V1.1_1.1.0_feed2055/ | previous stable 物料（嘉音Studio v1.1.0） |
| 当前/V1.1_1.1.1_feed2056/ | **当前 latest 发布物料（嘉音Studio v1.1.1）**：feed 2056（14 组件）、setup 1.1.1、manifest、SHA、release-notes |
| 历史/ | 旧快照/旧 feed/旧上传脚本（STALE/LEGACY 说明），只读不改 |

- 验证用 `local-mirror` feed 已随 F 盘测试环境清理而删除（重建方法见 `_接管审计/ops/make_local_mirror_feed.py`，需重新准备本地 runtime/资产副本）。
- 正式资产以 GitHub release 为准；本地 `当前/` 用于回滚、复核与离线安装。

## 5. F:/AMS-Studio 当前状态（2026-10-30 起）

用户已按 v1.0.0 setup 安装并试用；2026-10-30 处理了两项安装器问题：

- 已清空 `data/components/downloads`（v1.0.0 遗留的 35.4GB 暂存包，安装 59.31GB → 24.73GB）；
- 已删除重复的 `AMS-Studio-Uninstall.exe`，注册表 `UninstallString` 指向 `uninstall.exe`；
- 当前 launcher 仍为 **1.0.0**；建议运行 v1.0.1 setup 覆盖安装（只更新 launcher/安装器，保留 `data/` 用户数据），升级后 downloads 永久自动清空、不再产生第二个卸载器。
- 安装包位置：GitHub release `v1.0.1`，或本地 `09_发布/当前/H5.1_1.0.1_feed2053/AMS-Studio-Setup-1.0.1.exe`。
- 若 GitHub 大文件下载中断（已知问题），可重试 `ams-launcher.exe --install-components`；暂存残留可用 `ams-launcher.exe --cleanup-downloads` 清理（v1.0.1+）。

## 6. GitHub 仓库

| 仓库 | URL | 作用 |
|---|---|---|
| AMS-Studio | https://github.com/yan-mc/AMS-Studio | 顶层伞仓 + 子模块引用 |
| ams-launcher | https://github.com/yan-mc/ams-launcher | Launcher/Installer |
| ams-harness | https://github.com/yan-mc/ams-harness | DSH/Kernel 源码 |
| ams-services | https://github.com/yan-mc/ams-services | Gateway/VC |
| ams-plugins | https://github.com/yan-mc/ams-plugins | Host/Client/音乐 bundle |
| ams-tools | https://github.com/yan-mc/ams-tools | 工具集 |
| ams-release | https://github.com/yan-mc/ams-release | 发布/清单/签名 |
| ams-docs | https://github.com/yan-mc/ams-docs | 文档 |
| ams-agent-music | https://github.com/yan-mc/ams-agent-music | 独立 Agent 插件 |

### 6.1 伞仓结构

AMS-Studio 顶层：

- launcher/ -> ams-launcher
- kernel/ -> ams-harness
- plugins/ -> ams-plugins
- services/ -> ams-services
- tools/ -> ams-tools
- docs/ -> ams-docs
- release/ -> ams-release

## 7. 数据目录职责

| 路径 | 作用 |
|---|---|
| data/kernel | AMS Kernel/DSH 程序 |
| data/components | gateway/plugin/agent 组件 |
| data/runtime | RVC/TTS/pymss/tts-train runtime |
| data/datasets | 数据集与源素材 |
| data/voices | RVC 音源 |
| data/tts_voices | TTS 音源 |
| data/models | Vosk 等辅助模型 |
| data/projects | 用户工程 |
| data/artifacts | 任务产物 |
| data/config | 状态/连接配置 |
| data/logs | 运行日志/审计 |
| data/trash | 应用内回收站 |
| data/backups | 应用内备份 |

## 8. 当前隔离区（用户数据备份）

F:/_trash/ams-clean-20261026_1445（约 4.4GB，**用户数据，勿删**）

保留的资产：

- voices/曼波、voices/闲云、voices/xianyun_zh
- tts_voices/闲云、tts_voices/zh_CN-huayan-medium、tts_voices/en_US-lessac-medium
- models/vosk-cn（ASR 模型）
- tools/asr（Vosk 依赖，agent-music ASR 需要）
- datasets/xianyun640、datasets/tts
- projects、config、logs、qa、exports、assets

F 盘测试环境/测试残留已于 2026-10-26 17:25 清理（AMS-H4-H3 / AMS-H4-RC2 / AMS-H5-VERIFY / AMS-H2-VERIFY / AMS-BUILD-VERIFY / _gw-selftest* / _edge-cdp-* / _logs / _agent_music_test / _trash 内测试残留），明细见 `_接管审计/13_F盘清理与验证环境说明.md`。

另保留 `F:/AMS-keys-backup-20261026/`（Ed25519 签名密钥仓库外备份，2 文件）。

## 9. 重要提醒

- ams-harness 是 Junction，不要删除 _harness_full。
- 两个卸载文件不是重复安装：AMS-Studio-Uninstall.exe 是 NSIS 卸载器，uninstall.exe 是 launcher 识别别名。
- PowerShell 不要使用 $home 作为路径变量；用 $amsHome / $safeTarget。
- 不要直接递归删除 F:/_trash，先确认不再需要曼波/闲云/vosk 模型等本地资产。
- F 盘已无测试安装环境；用户全新安装时用 v1.0.0 setup，安装/回滚说明见 CURRENT_STATE 与 09_发布/README。
