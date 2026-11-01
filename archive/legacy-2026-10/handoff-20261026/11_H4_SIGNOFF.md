# H4 签核（正式）

日期：2026-10-26 17:05（Asia/Shanghai）  
候选：**AMS Studio v1.0.0-rc.2**，feed **2051**，GitHub release `v1.0.0-rc.2`（id 424012659）  
依据：`_接管审计/10_H4_验收报告.md`、`09_H3_验收报告.md` 及对应 `qa/` 证据目录

---

## 结论

**H4 通过，签字放行 H5 v1.0.0 发布。**

## 签字依据（全部有可复现证据）

| H4 验收项 | 结果 | 证据 |
|---|---|---|
| 候选冻结（版本/feed/哈希） | PASS | 10_H4 报告 §1 |
| feed 2051 签名有效 | PASS | `manifest.py verify` seq=2051、13 组件 |
| GitHub release 7 资产 digest 与本地一致 | PASS | 10_H4 报告 §8 |
| setup 干净安装 + launcher 内嵌前端资源 | PASS | 10_H4 报告 §2.1（launcher 6,901,760 B、5 个资源 key） |
| 从 feed 安装 13 组件 + 干净环境启动 | PASS | 10_H4 报告 §2.3（全 13 ok=true；Gateway health；Kernel 14930；14 页签 0 error） |
| UI 14 页签 CDP | PASS | `qa/ui-cdp-20261026-163803`（离线环境 14/14）、`qa/ui-cdp-20261026-165630`（干净环境 14/14） |
| GPU E2E 基础 6 项 | PASS | `qa/gpu-e2e-20261026-163702`（6/6，含 RVC 3min 26s、pymss 4 stems、TTS、评测、混音） |
| 12 分钟长音频 RVC | PASS | job_0b93342faba54561 |
| 训练链 dataset→train→注册→转换 | PASS | job_8f777a655e184cfb / job_043e252673514875 / job_4f48ab8bce844608 |
| H2→rc.2 升级 / rc.2→H2 回滚 / 卸载保留 data | PASS | 10_H4 报告 §3 |
| 20 tools + skill（含安全边界） | PASS | `data/logs/agent-music-load.log`（H4-H3 环境） |
| 无 C 盘 AMS data / 未认证 401 / token 不泄漏 | PASS | 05 环境恢复报告、10_H4 报告 §7 |

## 未全绿项与处置（不影响签字）

1. **严格远端 feed 全新安装 PARTIAL**：GitHub release-assets 在 kernel 388MB 下载中断（本次 84MB；历史 251MB）。处置：记为已知限制；用全本地镜像 feed 完成同一安装链路验证；H5 继续沿用 rc.2 起可用的"离线包/镜像 feed"方案，并在 09_发布 README 中写明回滚与离线安装方式。
2. **全量 xianyun640 1 epoch 训练中途被操作误杀**：已用小数据集完整重跑训练链 PASS，全量训练能力有 H2 50 epoch 历史证据；不影响 H4。

## 签字

- 签字人：接手负责人（Lead）
- 结论：**SIGNED / PASS**
- 放行：可以发布 H5 v1.0.0（feed 2052），并将 latest 从 v0.2.1 切换到 v1.0.0
