# H0 基线冻结（2026-10-25）

## 1. 冻结结论

H0 基线冻结完成。自此进入“功能冻结、只修复发布阻断”的阶段。

- 产品目标版本：v1.0.0
- 当前基线标签：h0-baseline-20261025
- 当前发布分支：release/1.0
- 冻结范围：阶段 A–F 核心功能 + 阶段 G 发布修复
- 不在 v1.0 范围：V 家填词、完整 DAW、强制插件拆分

## 2. 代码基线

| 模块 | 仓库 | commit |
|---|---|---|
| umbrella | AMS-Studio | tag:h0-baseline-20261025 |
| launcher | ams-launcher | e130e17 |
| kernel | ams-harness | b21fa26 |
| plugins | ams-plugins | 04461d5 |
| services | ams-services | 216607a |
| docs | ams-docs | tag:h0-baseline-20261025 |
| release | ams-release | 727c3e6 |
| tools | ams-tools | da17d45 |

## 3. 发布基线

- feed：2042
- setup：2,438,343 字节
- setup SHA256：D2DEFF5125CD857F41D9CE7A7C689E79A1DDA88AADA55F486DF4AE6060338B45
- service.gateway：16,809,048 字节，sha256 4a96d79e...
- plugin bundle：45,816 字节，sha256 3d5345d0...
- 已知元数据缺陷：feed 中 Gateway version 仍写 0.1.0-tts-import，代码已是最终版，H1 必须修正。

## 4. 环境基线

- Windows 10.0.26300
- PowerShell 5.1
- NVIDIA RTX 4080 Laptop GPU，驱动 621.01
- 安装目录：F:/AMS-Studio
- 数据目录：F:/AMS-Studio/data
- Node/pnpm：随项目工具链

## 5. 测试夹具

| 夹具 | 用途 |
|---|---|
| data/assets/preview_voice.wav | 音色试听 |
| data/projects/imports/e_audit_3min.wav | 3 分钟换源 |
| data/projects/imports/e_audit_12min_long.wav | 12 分钟分段 |
| data/datasets/ljspeech15 | 训练 14.8 分钟 |
| data/tts_voices/zh_CN-huayan-medium | 中文 TTS |
| data/tts_voices/en_US-lessac-medium | 英文 TTS |

完整哈希见 TEST_DATA_MANIFEST.csv。

## 6. 变更控制

冻结期间允许：

- 修复 P0/P1 缺陷；
- 版本号/feed/setup/manifest 一致化；
- 文档、测试、安装脚本；
- 不改变用户数据布局。

冻结期间禁止：

- 新功能；
- 大规模重构；
- 插件拆分；
- 填词/DAW 扩张；
- 未经清单确认的测试数据清理。

所有变更必须：

1. 关联缺陷 ID；
2. 通过对应验收；
3. 更新 docs/PROJECT_RECORD；
4. 更新 feed/setup 版本；
5. 保持 data 兼容。

## 7. 已知缺陷

见 DEFECT_REGISTER.md。H0 退出时 P0 允许未修复，但必须登记；进入 H1 后 P0 必须清零。

## 8. H0 退出条件

- 范围冻结；
- 版本/commit/feed/setup 记录；
- 测试夹具清单；
- 缺陷登记；
- release 分支与基线 tag；
- 项目总记录更新；
- GitHub 同步。

## 9. 下一阶段

H1：安装/升级/卸载/Kernel 生命周期。H1 开始前不进行功能开发。
