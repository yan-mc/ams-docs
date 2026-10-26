# H5 v1.0.0 发布报告

日期：2026-10-26 17:01–17:05（Asia/Shanghai）  
执行：接手负责人（Lead）  
前置：H4 签字（`11_H4_签字.md`）

---

## 1. 发布内容

| 项 | 值 |
|---|---|
| 产品版本 | **1.0.0** |
| GitHub release | **v1.0.0**（id 424030311，prerelease=false，**latest 已切换**） |
| feed sequence | **2052** |
| feed sha256 | `0DCF25E73FED7CDE0B58194B549E697144E213F2033438D2EB73E3B3F0776EF3` |
| setup | `AMS-Studio-Setup-1.0.0.exe`，3,126,682 B，sha256 `8275319B04A041C1249DD24BF4E9C741F3965F8E60AD1BCA3E2CAD127B64C4FA` |
| launcher | 6,901,760 B，sha256 `9447B6B6D26F0E26CEF6668C5F3E259E44F0A8D81BF71FC05FB4202C444D23EF`（内嵌前端资源） |
| gateway | `service-gateway-1.0.0.zip`，17,238,201 B，sha256 `9566CD153A07758151773E03981486F8AF5DB5E7A1CA1187749F7996AD7F2DD5` |
| plugin | `plugin-ams-music-bundle-1.0.0.zip`，46,140 B，sha256 `BA25180F3A9392CEC3E75CC964066518A4A62BAC51ABB2620F963A983D6720EC` |
| kernel | 0.2.1-rc.3（复用，388,093,624 B / `CE507367…`） |
| runtimes | rvc / pymss / tts / tts-train 0.1.0（Hugging Face，未变） |
| 源码 commits | launcher 8ae99a9 / plugins 0e211c8 / services dc8de91 / release 6c3324c |

## 2. v1.0.0 相对 rc.2 的改动

1. 版本号统一升到 **1.0.0**（launcher/installer/gateway/plugin）。
2. **修复 launcher "安装音乐插件到内核" 的硬编码 tgz 名**（rc.1/rc.2 写死 `*-0.3.0.tgz`，rc.2 组件改名后该按钮失效）→ 改为动态发现 `ams-plugin-core-* / ams-plugin-ui-* / ams-music-bundle-*.tgz`。
3. launcher UA 由硬编码 `ams-launcher/0.2.0` 改为 `CARGO_PKG_VERSION`。
4. 重新构建全部组件（setup/launcher/gateway/plugin），构建过程通过"内嵌前端资源"硬断言。

## 3. 发布后验证（已执行）

| 验证项 | 结果 |
|---|---|
| GitHub release 7 资产上传 + digest 回读 | PASS：全部与本地 sha256 一致 |
| `releases/latest` = v1.0.0 | PASS |
| `releases/latest/download/ams-components.json` | seq=2052、product=1.0.0、13 组件 |
| 远端 feed 验签（manifest.py verify --min-sequence 2052） | PASS（OK: signature valid, schema ok, sequence=2052） |
| 远端 feed sha 与本地 | 一致（`0DCF25E7…6EF3`） |
| H5 setup 干净安装 | PASS（`F:/AMS-H5-VERIFY`，launcher 1.0.0 / 6,901,760 B / 内嵌资源） |
| H5 干净安装组件（本地镜像 feed 2052） | **PASS：13/13 组件 ok=true**（kernel 1,041MB、runtime 23,717MB、components 35,442MB、tools/voices/tts_voices 齐全；`config/runtime.json` 正确写入） |
| H5 干净环境 Gateway | PASS：health ok=true，version=**1.0.0**，job_types 12 类（含 mix.master） |
| H5 干净环境 Kernel + UI | PASS：`launcher --start-kernel` 启动，UI CDP **14/14 PASS、0 console error**（`qa/ui-cdp-20261026-170920/`） |

## 4. 旧 release 处置（非破坏，资产全保留）

| tag | 新名称 | 说明 |
|---|---|---|
| v0.2.1 | AMS Studio v0.2.1 (previous stable) — feed 2049 | 回滚源；setup/launcher 已确认可用 |
| v1.0.0-rc.1 | AMS Studio v1.0.0-rc.1 (superseded) — feed 2050 | 已知 dev-mode launcher 缺陷 |
| v1.0.0-rc.2 | AMS Studio v1.0.0-rc.2 (superseded) — feed 2051 | H4 候选 |
| v0.2.0 | AMS Studio v0.2.0 (archived — superseded by v0.2.1) | 历史 |

## 5. 发布后仍存在的已知问题（H5.1/后续）

1. GitHub release-assets 大文件下载不稳定（kernel 388MB 会中断）→ 建议 H5.1 给 kernel 增加 HF 镜像或 launcher 分片续传。
2. 安装包/launcher 未做 Authenticode 代码签名（SmartScreen 提示）。
3. 生产私钥仍在仓库工作树（已备份 `F:/AMS-keys-backup-20261026/`），轮换需重签新公钥启动器。
4. `agent-music`（20 tools）未纳入 feed，需独立安装（发布说明中注明）。
5. H2 latest 已不再是 latest，但 v0.2.1 release 保留为 previous stable 回滚源。

## 6. 相关文件

- 发布物料：`09_发布/当前/H5_1.0.0_feed2052/`（setup/gateway/plugin/feed/manifest/SHA/notes + local-mirror）
- 发布 SOP：`09_发布/README.md`
- H4 验收/签字：`_接管审计/10_H4_验收报告.md`、`_接管审计/11_H4_签字.md`
- 当前事实：`CURRENT_STATE.md`
