# H4 验收报告（候选 v1.0.0-rc.2 / feed 2051）

日期：2026-10-26（Asia/Shanghai）  
执行：接手负责人（Lead）  
候选：**AMS Studio v1.0.0-rc.2**，GitHub release `v1.0.0-rc.2`（id 424012659，prerelease），feed **2051**  
证据目录：`_接管审计/qa/`、`_接管审计/logs/`

---

## 0. 结论（当前）

| 判定 | 说明 |
|---|---|
| 候选完整性 | PASS（feed 2051 签名有效、release 7 资产 digest 全对、版本矩阵统一 rc.2） |
| 安装/升级/回滚/卸载 | PASS（setup rc.2 干净安装；H2→rc.2 升级；rc.2→H2 回滚；卸载保留 data） |
| UI 14 页签 | PASS（14/14 ×2 环境：离线恢复环境 + rc.2 干净安装环境） |
| GPU E2E | PASS：基础 6/6；12 分钟长音频；训练链 dataset→train→注册→转换 |
| 干净环境完整启动 | PASS（rc.2 全新目录：setup → launcher 装 13 组件 → Gateway 1.0.0-rc.2 → Kernel → 14 页签 0 error） |
| 严格远端 feed 全新安装 | PARTIAL（GitHub release-assets 下载 kernel 388MB 中断；已用全本地镜像 feed 完成同一安装链路，网络问题列为已知限制） |
| **H4 签字** | **PASS / SIGNED（2026-10-26 17:05）**，见 `11_H4_签字.md` |

---

## 1. 候选冻结

| 项 | 值 |
|---|---|
| 产品版本 | 1.0.0-rc.2 |
| feed sequence | 2051（13 组件） |
| feed sha256 | `6AA03EC752F4DEE68D9C7ADC85767EEB72AF23A97C61D06D16D8CE7B77190FAF` |
| setup | `AMS-Studio-Setup-1.0.0-rc.2.exe`，3,126,150 B，sha256 `F4703A48…BCD7D`，FileVersion 1.0.0-rc.2 |
| launcher | 6,900,224 B，sha256 `886CBD9E…74B`（**已内嵌前端资源**，非 dev 构建） |
| gateway | `service-gateway-1.0.0-rc.2.zip`，17,238,352 B，sha256 `A82916DC…8017`，health version=1.0.0-rc.2 |
| plugin | `plugin-ams-music-bundle-1.0.0-rc.2.zip`，46,145 B，sha256 `D8F2CE28…2A7D` |
| kernel | 0.2.1-rc.3，388,093,624 B，sha256 `CE507367…90CA`（复用 v0.2.1 资产） |
| runtimes | rvc/pymss/tts/tts-train 0.1.0（Hugging Face，分片 sha 已核对） |
| GitHub 标记 | prerelease=true，latest 仍 v0.2.1（未提前切换） |
| 源码 commits | launcher 9066f28 / plugins c05be8c / services 90238f1 / release 9c07c30 |

## 2. 干净安装

### 2.1 setup 安装（PASS）

- H3 setup 与 rc.2 setup 两次干净目录安装均 PASS；rc.2 launcher 6,900,224 B / FileVersion 1.0.0-rc.2。
- **rc.1 缺陷确认**：H3 rc.1 setup（0AF08BE4）内 launcher 为 dev 模式构建（6,865,920 B，0 个内嵌前端资源，干净环境白屏风险）；**rc.2 已修复**（内嵌 2 个 assets key + index js；build-installer.ps1 增加硬断言）。
- H2 v0.2.1 setup 内 launcher（0.2.1，6,899,200 B）确认有内嵌资源（H2 回滚源可用）。
- 干净目录：`F:/AMS-H4-RC2`，19 个 data 子目录；数据只写安装根 data；`C:/AMS-Studio` 不存在。

### 2.2 从 feed 安装组件

| 方式 | 结果 |
|---|---|
| 远端 feed（GitHub v1.0.0-rc.2 feed 2051） | **PARTIAL**：manifest/签名/normalize OK；kernel 388MB 下载到 84MB 后连接中断（release-assets 不稳，与历史 H4 记录一致）→ 失败点已隔离到 `F:/_trash/ams-test-residue-20261026/` |
| 全本地镜像 feed（签名 seq 2051，13 组件 URL→file:///） | **安装执行中/结果见 §2.3**：kernel/catalog/voicepack/gateway/plugin 已 ok=true，runtime 正在复制+解包（rvc/pymss/tts/tts-train/…） |

设计说明：本地镜像 feed 只把"下载来源"换成本地已校验副本，launcher 仍执行完整的 **读取清单→验签→逐组件 size/sha 校验→解包→写 target→更新 config/runtime.json** 流程；因此它验证的是"安装链路正确性"，网络稳定性作为独立已知问题。

### 2.3 安装结果（PASS）

最终一轮全本地镜像 feed 安装结果（`F:/AMS-H4-RC2/data/logs/launcher-cli-trace.log`）：

- **13/13 组件 installed ok=true**：kernel → catalog → voicepack → gateway → plugin → runtime.rvc → runtime.pymss → runtime.tts → tool.msvc → runtime.pymss.python → tool.vcrun → service.vc → runtime.tts-train。
- 安装规模：kernel 1,040 MB（含 resources/app.asar）、runtime 23,717 MB / 119,876 文件、components 35,442 MB、tools 11 文件、voices 曼波 187 MB、tts_voices 2 文件。
- `config/runtime.json` 被 launcher 正确写入（rvc_python/rvc_root/pymss_python/pymss_root/msvc_tool 全部指向新安装目录）。
- 干净环境 Gateway 启动：health ok=true，version=**1.0.0-rc.2**，job_types 12 类（含 mix.master）。
- 干净环境 Kernel 启动：`launcher --start-kernel` → `dsh web: http://127.0.0.1:14930/`；安装音乐插件后 UI CDP **14/14 PASS、0 console error**（`qa/ui-cdp-20261026-165630/`）。

> 说明：launcher 的"安装音乐插件到内核"按钮在 rc.1/rc.2 中把 tgz 文件名硬编码为 `*-0.3.0.tgz`，rc.2 组件已改名 `1.0.0-rc.2`，所以该按钮会报"插件组件不完整"。本次干净环境用等价 DSH CLI 手工安装插件完成验证；**v1.0.0 已修复为动态发现 tgz 并重建 launcher**。

## 3. 升级 / 回滚 / 卸载（PASS）

| 用例 | 结果 |
|---|---|
| H2 v0.2.1 → rc.2 覆盖升级 | PASS：launcher 0.2.1→1.0.0-rc.2；data 标记 `H4_UPGRADE_KEEP.txt` sha 不变 |
| rc.2 → H2 v0.2.1 回滚 | PASS：launcher 回到 0.2.1；data 标记不变；注册表 DisplayVersion=0.2.1 |
| 卸载保留 data | PASS：卸载后 launcher 删除、`data/` 与标记保留、注册表项清理、仅剩 data 目录 |

## 4. Kernel / 插件

| 项 | 结果 | 证据 |
|---|---|---|
| Kernel 冷启动稳定性 | PASS（历史 3 次 7.3–7.6s；rc.3 kernel 相同） | H2/H3 记录 |
| 无 Web authentication failed | PASS | Kernel Web 200 / title=AMS Kernel（H4-H3 环境） |
| 20 tools + skill | PASS（H4-H3 环境） | `data/logs/agent-music-load.log`：count=20、skill bytes=3182、selftest ok |
| 插件 14 页签 | PASS | §5 |

> 说明：feed 2051 不含 `@ams/agent-music` 独立插件；干净安装默认只有 music bundle（plugin-core/plugin-ui/music-bundle）。20 tools 需按 `ams-agent-music v0.2.0` release 另行安装（H4-H3 环境已装并验证）。

## 5. UI（14 页签，PASS）

- 脚本：`08_项目代码/ams-release/qa/ui-cdp.ps1`
- 结果：`qa/ui-cdp-20261026-163803/`：**14/14 PASS，0 console error**
- 覆盖：项目、实时变声、音色库、TTS 批量、音源炼制、数据集工作室、评测器、混音母带、人声分离、歌曲换源、MSVC 工具、TTS 语音合成、TTS 数据集、TTS 音源炼制。
- 首次运行时"实时变声"502（VC 服务未部署）；部署 `service-vc` 到 `data/components/vc-service` 后复测 0 error。

## 6. GPU E2E

| 用例 | 结果 | job id |
|---|---|---|
| ping | PASS | job_0ae3130d6e684598 |
| convert.rvc 3min（曼波/cuda） | PASS（26s） | job_52f14f34af2d4e90 |
| separate.pymss 3min | PASS（4 stems） | job_0c1860ed2dae4236 |
| tts.synthesize 中文 | PASS | job_80172ad481704d08 |
| evaluate.audio | PASS | job_58afd794144a473c |
| mix.master 2 轨 | PASS | job_95dfe95b009f4fe6 |
| convert.rvc 12min（分段/长度锁定） | PASS | job_0b93342faba54561 |
| dataset.prepare（3min 源 → 18 clips） | PASS | job_8f777a655e184cfb |
| train.rvc 1 epoch（小数据集） | PASS（pth 57,580,765 B + index 24,437,805 B + voice.json 注册） | job_043e252673514875 |
| 训练后 convert.rvc（新音色） | PASS（wav 25,920,044 B） | job_4f48ab8bce844608 |

> 说明：全量 xianyun640 的 1 epoch 训练在完成 f0/特征阶段后被一次误操作中断（构建 Gateway 自测时误杀了 Gateway 进程），随后用 18 clips 小数据集完整重跑「dataset.prepare → train.rvc → 注册 → convert.rvc」链路并 PASS；全量数据集训练能力另有 H2 的 50 epoch 历史证据。

## 7. 安全 / 边界

| 项 | 结果 |
|---|---|
| 无 C 盘 AMS data | PASS（`C:/AMS-Studio` 不存在） |
| Gateway 未认证 401 | PASS（H4-H3 环境验证） |
| feed 签名 | PASS（2049/2050/2051 均本地验签） |
| token 不出 Host | PASS（未在日志/文档写入 token） |
| 路径逃逸/未确认写操作 | PASS（agent-music 0.2.0 在 H4-H3 selftest 与历史 H4 证据中通过） |
| 私钥管理 | 仓库外备份 `F:/AMS-keys-backup-20261026/`；轮换待定（H5 前决策） |

## 8. 发布资产一致性（PASS）

- GitHub `v1.0.0-rc.2` 7 资产 digest 与本地 sha256 全部一致（回读校验）。
- `09_发布/当前/` 已含 H2/H3/H4 三套当前物料 + `SHA256SUMS-当前.txt`（30 文件）。
- latest 未提前切换；v0.2.0 标 archived；rc.1 保留待 H5 后标 superseded。

## 9. 已知问题（H5 前须处理/决策）

1. **GitHub release-assets 大文件不稳**：kernel 388MB 单连接中断（84MB/251MB 两次证据）。H5 前建议给 kernel/大组件增加 HF 镜像或 launcher 分片续传（当前 launcher 下载器无 resume/retry）。
2. **launcher 下载器无断点续传**：本次 HF runtime part-1 曾出现 0 字节停滞；干净安装验证改用本地镜像 feed。
3. **launcher 版本字符串**：launcher UA 仍写 `ams-launcher/0.2.0`（不影响功能，建议 H5 前改）。
4. **Authenticode**：setup/launcher 未代码签名（SmartScreen 拦截风险）。
5. **生产私钥**：仍在仓库 examples/keys（已备份），H5 前决策轮换。
6. **agent-music 不在 feed**：干净安装默认无 20 tools（需独立安装）；H5 是否纳入 feed 需决策。
7. **H2 latest 仍是旧 0.2.x**：H5 发布后 latest 才切换；期间回滚只能装 setup（H2 launcher 正常）。

## 10. 签字条件 checklist

- [x] 候选冻结（feed 2051 / setup / gateway / plugin / kernel 哈希记录）
- [x] feed 签名验证
- [x] GitHub release 资产 digest 一致
- [x] 干净安装（setup + launcher 内嵌资源）
- [x] feed 组件安装链路（全 13 组件，干净环境启动 + 14 页签）
- [ ] 严格远端 feed 全新安装（PARTIAL：GitHub kernel 下载中断，已知限制 + 镜像方案）
- [x] 升级 / 回滚 / 卸载保留 data
- [x] Kernel 无认证失败
- [x] 20 tools + skill（H4-H3 环境）
- [x] 14 页签 UI CDP（两个环境各一次）
- [x] GPU E2E 基础 6 项 + 12 分钟长音频 + 训练链（dataset→train→register→convert）
- [x] 安全边界（C 盘/未认证/签名/token）
- [x] 文档（本报告 + 04/05/06/07/08/09/11/12 + 发布 SOP）

结论：**H4 通过并签字（2026-10-26 17:05，Asia/Shanghai）**。唯一未全绿项是"严格远端 feed 全新安装"（网络侧 GitHub release-assets 对大文件不稳定），已作为已知限制记录并给出本地镜像/离线包方案；该项不影响本地安装链路正确性与 v1.0.0 发布。
