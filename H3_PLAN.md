# H3 发布候选（RC）计划与预检（2026-10-25）

状态：**H3 预检完成，尚未进入 RC 执行。H2 中文闲云全量训练挂起。**

## 1. H3 目标

在 H2 核心回归完成后，产出一个可安装、可升级、可回滚、可验收的 **AMS Studio v1.0.0-rc.1** 发布候选。

建议 RC 版本：

    AMS Studio 1.0.0-rc.1

最终 v1.0.0 由 H5 发布。

## 2. H3 五项硬要求

依据总计划：

1. 自动化套件；
2. feed/setup 统一；
3. 文档；
4. 干净目录/便携 RC（VM 方案已废弃）；
5. 缺陷清零。

## 3. 当前信息核对

### 版本

| 组件 | 当前 |
|---|---|
| Launcher | 0.2.1 |
| Installer | 0.2.1 |
| Plugin | 0.3.0 |
| Gateway | 0.1.0 |
| feed | 2044 |
| docs | 0.2.0 / feed 2042 |

结论：版本未统一，H3 必须做 RC 版本矩阵。

### feed/setup

feed 2044 的 setup：

    AMS-Studio-Setup-0.2.1.exe
    3,125,583 bytes
    F6924345F5064529ADD5678DD72CC9ADF2A3757E921FA1B65BEAE42E486E6C05

GitHub v0.2.1 当前 RELEASE_MANIFEST 仍是：

    sequence 2043
    product_version 0.2.0
    setup 0.2.0 / D2DE...
    commits 旧基线

SHA256SUMS 也是旧内容，不含 0.2.1 setup。

结论：**feed/setup 发布清单不一致，是 H3 P0 阻断。**

### GitHub release 资产

v0.2.1 当前 62 个资产，含大量 stage/历史插件与 gateway 包：

- plugin-ams-music-bundle-0.3.0-stage*.zip
- service-gateway-0.1.0-stage*.zip
- service-vc-0.1.0-*.zip
- AMS-Studio-0.2.0-installlocal.msi

结论：H3 需要资产保留策略、旧包归档、SHA256SUMS 重建。

### 自动化

Gateway pytest：

    41 项通过

Release manifest 测试：

    1 项失败：ModuleNotFoundError: cryptography

原因：QA 环境不是自包含的，release 测试使用了缺少 cryptography 的 venv。

现有自动化：

- Gateway pytest：可用；
- H2 音频回归：ad-hoc 脚本；
- UI：Edge CDP 脚本，临时放在 data/logs；
- 安装器：手工测试；
- 干净 VM：方案已废弃，改为干净目录/便携 RC。

结论：**没有单一 H3 QA 命令，是 H3 P1 阻断。**

### 干净 VM（已废弃）

2026-10-25 已彻底拆除 `F:\AMS-VM`：VHDX、Windows ISO、Autounattend、Hyper-V 配置、pycdlib 工具及空目录全部删除；`F:\` 仅保留 `F:\AMS-Studio`。

原因：虚拟机无法正常调用 RTX 4080，VM 内无法完成 RVC/pymss/VC 的真实 GPU 验收。

H3 替代方案：

- 干净目录安装 RC（`F:\AMS-Studio-RC-Clean` 或新的空白目录）；
- 便携 RC 包；
- 如必须真正全新系统，使用独立物理机，不再用无 GPU 的 VM。

### 文档

INSTALLATION.md 仍写：

- Launcher 0.2.0；
- feed 2042；
- setup 0.2.0。

RELEASE_2042.md 仍是旧发布说明。

结论：H3 文档必须更新，否则 RC 不可交付。

## 4. H3 工作分解

### A. 版本与冻结

- 决定 RC 版本为 1.0.0-rc.1；
- launcher/installer/plugin/gateway 版本统一；
- 更新 package/Cargo/tauri/plugin/gateway 元数据；
- 生成 RC 版本矩阵；
- 创建 release/1.0-rc 分支；
- 冻结功能，只修 P0/P1。

验收：所有组件报告 1.0.0-rc.1，或明确映射表。

### B. 自动化套件

新增单一入口，例如：

    qa/run-h3.ps1

必须覆盖：

1. 版本矩阵校验；
2. feed 签名与 sequence；
3. release manifest 与 setup 哈希；
4. Gateway pytest（41 项）；
5. release manifest tests；
6. Host 路由/错误码；
7. UI 12 页签 CDP smoke；
8. 安装/升级/卸载/保留 data；
9. TTS/evaluate/mix 音频 smoke；
10. 干净目录/便携 RC 安装 smoke（RC 必跑）。

输出：

    data/qa/H3/<timestamp>/results.json
    data/qa/H3/<timestamp>/summary.md

先修：

- release 测试环境补 cryptography；
- 所有脚本固定 AMS_HOME；
- 不依赖临时 data/logs 脚本。

### C. 发布包统一

- 用独立安装器重建 1.0.0-rc.1 setup；
- 构建 launcher 1.0.0-rc.1；
- 构建 plugin/gateway RC 组件；
- feed 新 sequence（建议 2045）；
- 生成 RELEASE_MANIFEST.json 和 SHA256SUMS.txt；
- Ed25519 签名并 verify；
- feed/setup/launcher/plugin/gateway 同版本；
- 资产保留策略：
  - 保留当前 RC；
  - 保留一个可回滚旧版；
  - 其余 stage 包归档，不删除源材料。

验收：从 GitHub 下载 setup + feed，哈希/签名/版本一致。

### D. 干净目录/便携 RC

- 使用全新空白目录 `F:\AMS-Studio-RC-Clean`；
- 安装 RC setup；
- 启动 Kernel/Gateway/UI；
- 检查无 C 盘 AMS data；
- 从 0.2.1 升级到 RC，确认 voices/projects/config 不变；
- 卸载 RC，确认 data 保留；
- 回滚到 0.2.1（如回滚包存在）；
- 记录目录快照/文件清单和证据。

验收：全新目录 30 分钟内启动；升级/卸载/回滚 data 安全。

### E. 文档

- INSTALLATION.md 更新到 RC；
- RC 发布说明；
- SHA256/签名验证说明；
- 升级/回滚/离线安装；
- 已知问题与边界；
- 数据目录与安全边界；
- 故障排查；
- 下载链接与版本矩阵。

### F. 缺陷清零

当前必须关闭/明确：

- H2 中文闲云训练与最终 RVC（挂起，训练完成后再做）；
- pymss runtime.json 生成逻辑（当前手工修 .venv）；
- Gateway loopback 无 token 本机访问（安全评审或修复）；
- Gateway 组件 feed 版本（当前本地已重建，feed 仍旧）；
- 中文歌声听感（如无数据集，需明确 RC 边界）；
- VC 真实麦克风/RVC 长稳（passthrough 已过，真实音频未验）。

P0/P1 未清零不得签 RC。

### G. 回滚与离线

- 保存上一版 setup/feed；
- 回滚演练；
- 离线组件包；
- 更新失败恢复；
- CDN 固定 URL/缓存策略。

## 5. H3 验收清单

| 编号 | 项目 | 通过标准 |
|---|---|---|
| H3-1 | 版本统一 | 所有 RC 组件同版本/映射明确 |
| H3-2 | 自动化 | 单一命令，结果 JSON，失败可定位 |
| H3-3 | feed/setup | 签名、哈希、manifest、setup 一致 |
| H3-4 | 干净目录/便携 RC | 安装/启动/升级/卸载/回滚全通过 |
| H3-5 | 文档 | 安装/升级/回滚/已知问题完整 |
| H3-6 | 缺陷 | P0/P1 清零，P2 有豁免记录 |
| H3-7 | 数据安全 | 无 C 盘 AMS data，卸载保留 data |
| H3-8 | 回滚 | 可回到上一版 |
| H3-9 | 离线 | 无网络可安装核心包 |
| H3-10 | 发布物 | GitHub release 可复现 |

## 6. 建议顺序（1 周）

Day 1：版本冻结、P0/P1 清单、release manifest 修正方案。

Day 2：QA 环境自包含、release 测试修复、统一 QA 入口。

Day 3：构建 RC 组件/setup、签名、feed 2045、SHA256SUMS。

Day 4：干净目录/便携 RC 安装/升级/卸载/回滚。

Day 5：文档、离线包、资产保留策略。

Day 6：全量 QA、缺陷清零、RC 复测。

Day 7：RC tag、发布候选清单、H3 签收。

## 7. H3 入口条件

- H2 中文闲云训练完成并通过最终 RVC；
- H2 未完成项关闭或豁免；
- RC 版本决策；
- 干净目录/便携 RC 方案确认；
- 旧版回滚包确认。

## 8. H3 当前结论

H3 预检完成，但**不能直接进入 RC**。

最先处理的三个 P0：

1. RELEASE_MANIFEST/SHA256SUMS 与 feed 2044/setup 0.2.1 不一致；
2. 版本矩阵未统一；
3. QA 环境/release 测试不自包含（cryptography 缺失）。

随后处理：

- 干净目录/便携 RC；
- 自动化入口；
- 文档/回滚/离线；
- 缺陷清零。

H2 中文训练完成后，再恢复 H2 最后一项；H3 可并行做版本/自动化/发布清单，但 RC 签收必须等 H2 关闭。
