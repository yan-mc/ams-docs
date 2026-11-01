# 07 GitHub 发布整理与执行记录

日期：2026-10-26（Asia/Shanghai）  
执行：接手负责人（Lead）  
范围：`yan-mc/ams-release` 的 3 个历史 release + 新建 H4 候选 release；本文件记录**实际执行的写操作**（与 04 只读审计报告互补）。

---

## 1. 已有 release 元数据归一（2026-10-26 16:17）

| tag | id | 操作前名称 | 操作后名称 | 资产变动 |
|---|---|---|---|---|
| v0.2.1（latest） | 418556500 | `AMS Studio 0.2.0 + AMS Kernel 0.2.1-rc.2 (install-local)` | `AMS Studio v0.2.1 (latest) — feed 2049` | 无（11 资产） |
| v1.0.0-rc.1（prerelease） | 423803654 | `AMS Studio v1.0.0-rc.1` | `AMS Studio v1.0.0-rc.1 (prerelease) — feed 2050` | 无（6 资产） |
| v0.2.0 | 418450183 | `AMS components and update feed 0.2.0` | `AMS Studio v0.2.0 (archived — superseded by v0.2.1)` | 无（6 资产） |

- 正文同步更新为"版本 + feed + setup/sha + kernel + 用途/限制"结构；latest 判定未变（v0.2.1）。
- 操作方式：GitHub REST `PATCH /repos/yan-mc/ams-release/releases/{id}`；未删除/未改动任何资产。
- 备份：操作前快照见 `_接管审计/logs/github-releases-raw-20261026b.json`。

## 2. 新建 H4 候选 release（2026-10-26 16:43）

| 项 | 值 |
|---|---|
| tag | `v1.0.0-rc.2` |
| release id | 424012659 |
| 类型 | prerelease=true（latest 仍 v0.2.1） |
| 资产（7） | setup 3,126,150 / `F4703A48…`；gateway 17,238,352 / `A82916DC…`；plugin 46,145 / `D8F2CE28…`；RELEASE_MANIFEST 735 / `AE57C2A3…`；ams-components 9,848 / `6AA03EC7…`；SHA256SUMS 479 / `DC91F9C0…`；release-notes 241 / `24123813…` |
| 上传顺序 | setup → gateway → plugin → manifest → feed（最后）→ SHA256SUMS → release-notes |
| 回读校验 | 7/7 资产 digest 与本地 sha256 一致 |

- feed 2051 中 kernel/catalog/voicepack/service.vc 仍引用 v0.2.1 资产（未重复上传），runtimes 引用 HF；只有 setup/gateway/plugin 是本 release 新资产。
- 该 release 是 H4 验收候选；H4 通过后发布 **v1.0.0**（latest 切换），再回头把 v0.2.1 标成 previous stable、rc.1 标 superseded。

## 3. 资产保留/归档策略（执行口径）

| 组 | 资产 | 处理 |
|---|---|---|
| v0.2.1 必需 | feed / setup / kernel rc.3 / gateway stageg2 / plugin stageg2 / catalog / voicepack / service-vc / manifest / SHA | **保留，禁止删除**（H2 latest 回滚 + H4 feed 2051 交叉引用） |
| v0.2.1 回滚备件 | kernel rc.2（388.5MB） | 保留（至少保留到 H5 发布后） |
| v0.2.0 | 6 资产 | 已标 archived；保留不删（历史/审计） |
| v1.0.0-rc.1 | 6 资产 | 保留；H5 发布后正文标 superseded（资产不删） |
| v1.0.0-rc.2 | 7 资产 | H4 候选；保留 |
| H5 v1.0.0 | 新增 | 发布后成为 latest；旧 release 只改文案不删资产 |

## 4. 未执行/待确认

1. **未删除任何 GitHub 资产**（含 rc.2 kernel、v0.2.0 全部资产）。
2. **未改动 tag**（资产 URL 依赖 tag，改名断链）。
3. Authenticode 代码签名未做（无证书；H5 前待用户决策）。
4. 生产私钥轮换未做（当前仍用仓库内 `examples/keys/private.pem`；已在仓库外备份）。
5. H5 发布操作需用户最终授权（见 CURRENT_STATE §7）。

## 5. 相关文件

- 只读审计与建议：`_接管审计/04_发布整理报告.md`
- 发布 SOP：`09_发布/README.md`
- 当前事实：`CURRENT_STATE.md`
- 本地发布包：`09_发布/当前/H4_1.0.0-rc.2_feed2051/`
