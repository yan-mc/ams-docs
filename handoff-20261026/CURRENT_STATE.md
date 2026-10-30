# AMS Studio 当前状态（唯一事实来源）

更新：2026-10-26 17:10（Asia/Shanghai）  
维护：接手负责人（Lead）  
规则：**版本号、feed、哈希、发布状态以本文为准**；其他文档里与之冲突的数字一律视为历史快照（`11_验收与发布记录/` 等已加横幅）。本文只写已核实事实，每条都可用文件哈希或 GitHub release digest 复现。

---

## 0. 一页速查

| 项 | 值 |
|---|---|
| **稳定版 / GitHub latest** | **v1.1.0「嘉音Studio」，feed 2055**（2026-10-31 01:10 发布，prerelease=false；品牌改版 + Agent 插件一键安装） |
| Previous stable（回滚源） | v1.0.2（feed 2054）、v1.0.1（feed 2053）、v1.0.0（feed 2052）、v0.2.1（feed 2049） |
| Superseded（历史） | v1.0.0-rc.1（feed 2050）、v1.0.0-rc.2（feed 2051）、v0.2.0（archived） |
| H4 状态 | **PASS / SIGNED**（2026-10-26 17:05，见 `_接管审计/11_H4_签字.md`） |
| H5 状态 | **已发布**（见 `_接管审计/12_H5_发布报告.md`） |
| H5.3 / v1.1 状态 | **已发布**：v1.1.0「嘉音Studio」（品牌改版 + Agent 插件 0.2.1 一键安装 + 内核换标 client，见 `_接管审计/16`） |
| 版本矩阵 | launcher/installer = **1.1.0（嘉音Studio）**；gateway/plugin = **1.0.0**；kernel = **0.2.1-rc.3**；agent-music = **0.2.1**；runtimes/tools = 0.1.0 |
| 当前验证环境 | **无测试环境**；用户本机 `F:/AMS-Studio` 为 v1.0.1 安装（插件已装到内核，v1.0.2 只修显示；覆盖安装即升级） |
| 全新安装包 | GitHub release `v1.1.0`，或本地 `09_发布/当前/V1.1_1.1.0_feed2055/AMS-Studio-Setup-1.1.0.exe`（3,438,229 B / `1DCE2F79…E0BA`） |
| 唯一真实数据副本 | `F:/_trash/ams-clean-20261026_1445/AMS-Studio/data`（4.39GB / 曼波/闲云/tts 音源/数据集/工程/vosk 模型） |
| 生产签名私钥 | `ams-release/examples/keys/private.pem`（仓库内原件；仓库外备份 `F:/AMS-keys-backup-20261026/`） |
| 源码基线 | 8 仓库 main、clean；launcher **fe3c190** / agent-music **0bd5641** / plugins 0e211c8 / services dc8de91 / release 6c3324c |

---

## 1. 版本矩阵

| 组件 | v1.0.0（previous） | v1.0.1（previous） | v1.0.2（previous） | **v1.1.0（latest，嘉音Studio）** |
|---|---|---|---|---|
| launcher / installer | 1.0.0 | 1.0.1 | 1.0.2 | **1.1.0** |
| kernel（DSH fork） | 0.2.1-rc.3 | 0.2.1-rc.3 | 0.2.1-rc.3 | **0.2.1-rc.3**（独立版本线） |
| gateway | 1.0.0 | 1.0.0 | 1.0.0 | **1.0.0**（未变） |
| plugin bundle | 1.0.0 | 1.0.0 | 1.0.0 | **1.0.0**（未变） |
| **agent-music（独立，now in feed）** | — | — | — | **0.2.1**（20 tools + Skill + 内核换标 client） |
| runtime.rvc / pymss / tts | 0.1.0 | 0.1.0 | 0.1.0 | 0.1.0 |
| runtime.tts-train | 0.1.0 | 0.1.0 | 0.1.0 | 0.1.0 |
| tool.msvc / tool.vcrun / service.vc | 0.1.0 | 0.1.0 | 0.1.0 | 0.1.0 |
| catalog.models / voicepack.manbo | 2026.10.19 | 同 | 同 | 同 |
| feed sequence | 2052 | 2053 | 2054 | **2055** |
| feed sha256 | `0DCF25E7…6EF3` | `78601EB6…178D` | `1E251742…3264` | **`43AD98634E22A773415CEEC66D770A4219D41C2272C35EDA7E974A0A35A382D1`** |
| feed 组件数 | 13 | 13 | 13 | **14** |

---

## 2. GitHub 发布现状（yan-mc/ams-release，2026-10-31 01:10 实测）

| tag | release id | 名称 | latest | prerelease | feed | 资产数 |
|---|---|---:|---|---|---|---:|
| **v1.1.0** | 429294171 | 嘉音Studio v1.1.0 (latest) — feed 2055 | **true** | false | 2055 | 5 |
| v1.0.2 | 429214140 | 嘉音Studio v1.0.2 (previous stable) — feed 2054 | false | false | 2054 | 5 |
| v1.0.1 | 429165374 | AMS Studio v1.0.1 (previous stable) — feed 2053 | false | false | 2053 | 5 |
| v1.0.0 | 424030311 | AMS Studio v1.0.0 (previous stable) — feed 2052 | false | false | 2052 | 7 |
| v0.2.1 | 418556500 | AMS Studio v0.2.1 (older stable) — feed 2049 | false | false | 2049 | 11 |
| v1.0.0-rc.2 | 424012659 | AMS Studio v1.0.0-rc.2 (superseded) — feed 2051 | false | true | 2051 | 7 |
| v1.0.0-rc.1 | 423803654 | AMS Studio v1.0.0-rc.1 (superseded) — feed 2050 | false | true | 2050 | 6 |
| v0.2.0 | 418450183 | AMS Studio v0.2.0 (archived — superseded by v0.2.1) | false | false | 2000 时代 | 6 |

### v1.1.0（latest）资产（5）

| 资产 | 字节 | SHA256 |
|---|---:|---|
| ams-components.json（feed 2055，14 组件） | 10,326 | `43AD9863…82D1` |
| AMS-Studio-Setup-1.1.0.exe | 3,438,229 | `1DCE2F79…E0BA` |
| RELEASE_MANIFEST.json | 739 | `10FC1310…89CE` |
| SHA256SUMS.txt | 270 | `A41EB718…AE9B` |
| release-notes.md | 1,022 | `8C838111…52A0` |

- v1.1.0 = 品牌改版（嘉音Studio、圆形 logo、窗口/安装器名称、说明面板）+ Agent 插件 0.2.1 一键安装 + 内核换标 client；gateway/plugin 仍 1.0.0（复用 v1.0.0 资产），kernel 复用 v0.2.1 资产，runtimes 走 HF，agent-music 走独立仓库 release。
- 所有旧 release 资产保留（零删除）。

### 独立插件仓库 yan-mc/ams-agent-music

| tag | 资产 | 字节 | SHA256 |
|---|---|---:|---|
| **v0.2.1** | ams-agent-music-0.2.1.zip（Host 20 tools + Skill + @ams/agent-music-ui 换标 client + tgz 包） | 131,221 | `36B876F7…0AAB` |
| v0.2.0 | ams-agent-music-0.2.0.zip（历史） | 21,813 | `6117C69E…D14A` |

### 独立插件仓库 yan-mc/ams-agent-music

| tag | 资产 | 字节 | SHA256 |
|---|---|---:|---|
| v0.2.0 | ams-agent-music-0.2.0.zip | 21,813 | `6117C69E…D14A` |

---

## 3. 本地文件位置

| 内容 | 位置 |
|---|---|
| **当前有效发布物料** | `09_发布/当前/`：`H2_0.2.1_feed2049/`、`H3_1.0.0-rc.1_feed2050/`、`H4_1.0.0-rc.2_feed2051/`、`H5_1.0.0_feed2052/`、`H5.1_1.0.1_feed2053/`、`H5.2_1.0.2_feed2054/`、**`V1.1_1.1.0_feed2055/`**（各含 feed/setup/组件/manifest/SHA/unsigned/notes） |
| 历史/旧快照/旧脚本 | `09_发布/历史/`（含 STALE/LEGACY 说明） |
| 发布 SOP + 版本矩阵 + 资产映射 | `09_发布/README.md` + `SHA256SUMS-当前.txt` |
| 交接设计/结构/后续计划 | `PROJECT_DESIGN.md`、`PROJECT_FILE_MAP.md`、**`NEXT_STEPS.md`** |
| 接管审计/验收/签字/清理 | `_接管审计/00 接管方案`、`01-03 审计`、`04 发布整理`、`05 环境恢复`、`06 构建复现`、`07 GitHub 执行记录`、`08 文档迁移`、`09 H3 验收`、`10 H4 验收`、`11 H4 签字`、`12 H5 发布`、`13 F 盘清理与验证环境说明`、`14 下载缓存与安装器重复问题`、`15 插件状态判定与 agent-music 现状`、**`16 嘉音Studio v1.1 改版与 agent 插件`** |
| 源码 | `08_项目代码/`（8 仓库，main） |
| 伞仓（不可作交付基线） | `AMS-Studio/`（submodule 仍 pin 在 0.2.0 时代，未更新） |
| 签名密钥备份（仓库外） | `F:/AMS-keys-backup-20261026/`（private.pem sha `6A596F49…8E6E`） |
| 测试环境 | **无**（`F:/AMS-H4-H3` 等已删除；临时 `F:/_ams-fix-test`、`F:/_ams-plugin-test` 验证后删除） |
| 用户当前安装 | `F:/AMS-Studio` v1.0.1（音乐插件已安装到内核；v1.0.2 只修显示，覆盖安装即升级） |
| 真实数据（唯一副本） | `F:/_trash/ams-clean-20261026_1445/AMS-Studio/data/`（4.39GB，勿删） |

---

## 4. 状态与门禁

| 阶段 | 状态 | 证据 |
|---|---|---|
| H0 基线冻结 | PASS | `11_验收与发布记录/H0_*` |
| H1 安装/生命周期 | PASS | `11_验收与发布记录/H1_RESULTS.md` |
| H2 核心功能回归 | PASS | `11_验收与发布记录/H2_RESULTS.md` |
| H3 RC 发布 + UI/CDP + GPU E2E | PASS（本次补齐） | `_接管审计/09_H3_验收报告.md` |
| H4 验收签字 | **PASS / SIGNED** | `_接管审计/10_H4_验收报告.md`、`11_H4_签字.md` |
| H5 v1.0.0 发布 | **已发布**（v1.0.0 → previous stable） | `_接管审计/12_H5_发布报告.md` |
| H5.1 v1.0.1 补丁发布 | **已发布**（previous stable） | `_接管审计/14_下载缓存与安装器重复问题_20261030.md` |
| H5.2 v1.0.2 补丁发布 | **已发布** | `_接管审计/15_插件状态判定与agent-music现状_20261030.md` |
| v1.1.0 嘉音Studio 改版 | **已发布，latest=v1.1.0** | `_接管审计/16_嘉音Studio_v1.1改版与agent插件_20261031.md` |

---

## 5. 验收证据索引（`_接管审计/qa/`；测试环境已清理，证据文件保留）

| 证据 | 内容 |
|---|---|
| `ui-cdp-20261026-163803/` | H3/H4 UI 14 页签（离线恢复环境，14/14 PASS + 14 截图） |
| `ui-cdp-20261026-165630/` | rc.2 干净安装环境 UI 14 页签（14/14 PASS） |
| `ui-cdp-20261026-170920/` | **v1.0.0 干净安装环境 UI 14 页签（14/14 PASS，0 console error）** |
| `gpu-e2e-20261026-163702/` | GPU E2E 基础 6 项（6/6 PASS） |
| `gpu-e2e-20261026-164051/` | 扩展 E2E：12min 长音频 PASS；训练链另见 job 记录（job_8f777a655e184cfb / job_043e252673514875 / job_4f48ab8bce844608） |
| `20261026-164616/` | H4 QA 总脚本 rc.2（FAIL=0：版本矩阵/feed 验签/哈希/pytest/gateway health） |

复现命令见 `_接管审计/09_H3_验收报告.md` §6 与 `08_项目代码/ams-release/qa/README.md`。  
注意：截图/results 等证据仍在项目内；对应测试安装环境已删除（见 `_接管审计/13_F盘清理与验证环境说明.md`），复现需先按 `NEXT_STEPS.md` 重新安装环境。

---

## 6. 已知问题与后续（详见 `NEXT_STEPS.md`）

1. **GitHub release-assets 大文件不稳**：kernel 388MB 单连接会中断（84MB/251MB 两次证据）；launcher 下载器无断点续传。建议：kernel 增加 HF 镜像或 launcher 分片续传；离线安装包方案。**未修（P0-1）**。
2. **Authenticode**：setup/launcher 未代码签名，SmartScreen 会提示。**未修（P0-2）**。
3. **生产私钥**：仍在仓库 examples/keys（已仓库外备份）；轮换需重签内置新公钥的 launcher。**待决策**。
4. **agent-music 不能"下载即用"**：实测可手工安装并加载 20 tools + skill，但 ① 审计日志硬编码 `F:/AMS-Studio/data/logs/agent-music-load.log`；② ASR 依赖路径与实际 runtime 布局不一致（实际 `runtime/tts-train/0.1.0/runtime.tts-train-0.1.0/python`）；③ `models/vosk-cn` 与 `tools/asr` 不在 feed；④ 未纳入 feed/launcher 一键安装；⑤ `engines.dsh` 声明 rc.2（实际 rc.3，未阻塞）。**未修（P0-3）**。
5. **伞仓 `AMS-Studio` submodule 指针陈旧**（0.2.0 时代），不作为交付基线；交付以 08_项目代码 main + 发布包为准。
6. **H4 严格远端 feed 安装**：网络侧 PARTIAL（本地镜像链路 PASS），已记录为已知限制。
7. **TTS 训练质量**：音素 pad 已修、链路可用；听感调优仍按用户要求封存。
8. **旧数据迁移文档**：真实音源/数据集/vosk 模型仍在 `F:/_trash/ams-clean-20261026_1445`，用户新装后需按文档导入（见 `_接管审计/13` §5）。**未写（P0-4）**。
9. ~~**下载缓存重复**~~：v1.0.0 的 `data/components/downloads` 永久保留分卷+zip（全量安装约 35.4GB 冗余）→ **v1.0.1 已修**（分卷合并即删、解包即删、启动自愈、`--cleanup-downloads`）。见 `_接管审计/14`。
10. ~~**双卸载器**~~：v1.0.0 setup 同时生成 `AMS-Studio-Uninstall.exe` + `uninstall.exe` → **v1.0.1 已修**（只写 `uninstall.exe`，升级自动删旧文件）。
11. ~~**F 盘测试目录/旧安装**~~：已于 2026-10-26 清理完毕；用户 v1.0.0 安装的 downloads 与重复卸载器也已于 2026-10-30 清理修正。
12. ~~**音乐插件显示"未安装"**~~：`component_status()` 硬编码 `0.3.0` tgz 文件名，v1.0.x 的 `*-1.0.0.tgz` 永远判不到 → **v1.0.2 已修**（动态发现 + 区分"已下载/已安装到内核" + `--component-status` 诊断）。用户真实安装已验证显示"已安装到内核"。见 `_接管审计/15`。
13. ~~**agent-music 未完成**~~：v0.2.1 已修复日志/ASR 路径、进入 feed、启动器一键安装，并新增内核换标 client；详见 `_接管审计/16`。剩余：ASR 模型/Vosk 依赖仍不在 feed（可选自备），20 tools 自动化回归待做（P1-6）。

14. ~~**品牌/界面不一致**~~：启动器名/图标/内核页面仍为旧 AMS/DeepSeek 品牌 → **v1.1.0 已改**（启动器与安装器=嘉音Studio + 圆形 logo；内核通过 @ams/agent-music-ui 运行时替换标题/logo/文字/favicon，未改内核源码）。
15. **Kernel 可执行文件图标（任务栏/exe）无法由插件修改**：属于 AMS Kernel.exe 资源，需要重编内核；插件只改页面内视觉与窗口标题。

---

## 7. 维护规则

1. 任何版本/feed/哈希变化，先改本文件，再改其他文档；其他文档不得另立数字。
2. 发布物料只从 `09_发布/当前/` 取；`09_发布/历史/` 只作回滚与审计，不参与发布。
3. GitHub release 的标题/正文/资产变更必须在 `09_发布/README.md` 与 `_接管审计/07_GitHub_发布整理与执行记录.md` 记录。
4. 旧文档中的历史数字保留原文但已加"历史快照，见 CURRENT_STATE.md"横幅。
5. 发新版本流程：改版本号 → 构建 → 生成 feed（sequence+1）→ 签名验签 → 上传资产（feed 最后）→ 回读 digest → 干净安装验证 → 更新本文件与 09_发布 README。
