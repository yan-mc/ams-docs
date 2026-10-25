# H3 资产保留与归档策略（2026-10-25）

## 当前情况

GitHub v0.2.1 release 目前有 62 个资产，包含大量历史 stage 包：

- plugin-ams-music-bundle-0.3.0-stage*.zip
- service-gateway-0.1.0-stage*.zip
- service-vc-0.1.0-*.zip
- AMS-Studio-0.2.0-installlocal.msi
- 旧 setup/feed/manifest

## 保留原则

H3 发布前不做删除，只做归档/标记。保留：

1. 当前 H2 latest：feed 2044 / setup 0.2.1；
2. 当前 H3 RC draft：v1.0.0-rc.1 / feed 2045 / setup 1.0.0-rc.1；
3. 一个可回滚旧版：0.2.1 setup + 对应 feed；
4. Kernel/runtime 依赖资产；
5. 当前验证过的 voicepack/catalog。

可归档（先不删除）：

- 所有 stage* 插件包，仅保留最终 stageg2 作为 H2 基线；
- 所有 stage* gateway 包，仅保留 stageg2 与 RC 包；
- 历史 service-vc 迭代包，仅保留当前可用版本；
- 0.2.0 MSI/旧 setup（保留一个旧版回滚即可）；
- 重复的 manifest/checksums 旧资产。

## 执行时机

- H3 RC 验证完成前：不删除任何资产；
- H3 签收后：将 stage 包移到归档 release/tag 或本地归档；
- H4 前：只保留 RC、H2 latest、回滚版和运行依赖；
- 所有删除操作列出文件名/哈希/原因，逐项确认。

## 待办

- 生成 62 资产完整清单；
- 标记保留/归档；
- 生成新的 SHA256SUMS（只覆盖保留资产）；
- 更新 RELEASE_MANIFEST；
- H3 发布说明写明回滚版本。
