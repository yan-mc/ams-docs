# M0 验证：MSVC → RVC v2 .pth 转换器

> 时间：2026-10-19｜对象：心月-变声器 `_internal/modules/*/*.msvc`（本机 3 个）
> 工具：`08_项目代码/ams-tools/msvc2pth/`｜验证环境：`F:/ai歌曲/RVC20260311/runtime/python.exe`（torch 2.0.0）

## 1. 结论

| 模型 | 容器/张量表 | 457 张量键集 | 生成 pth | RVC load+infer | 状态 |
|---|---|---|---|---|---|
| 爱莉希雅（48k） | ✅ | ✅ | ✅ 57.6MB | ✅ unexpected=0，missing 全 enc_q，infer 有限 | **PASS** |
| 法国赌神（40k） | ✅ | ✅ | 可生成但数据分片未对齐 | ❌ | **WIP**：数据边界校准中 |
| 曼波Pro（48k） | ✅ | ✅ | 可生成但数据分片未对齐 | ❌ | **WIP**：数据边界校准中 |

**已证明**：MSVC 不是“改架构/加密权重”，只是容器混淆（14B 头 + 3 条反转 TLV + 16B trailer；metadata + 张量表 JSON + 明文 FP16 + 分片重排）；可还原为标准 RVC v2 `.pth` 并推理。

## 2. 爱莉希雅转换结果

- 输入：`F:/ai配音/心月-变声器/心月变声/_internal/modules/爱莉希雅/爱莉希雅.msvc`（57,748,698 B，sha256 `308d84a9…`）
- 输出：`07_执行计划/验证/msvc2pth_out/爱莉希雅/爱莉希雅.pth`（57,580,765 B）
- 报告：`conversion.json`（JSON 片段顺序 `[1,3,0,4,2]`、trims `[2,1,1,0,0,0,1,1,0]`、数据分片 `[2508,997,57428987,999,497]`、分片顺序 `[2,4,0,1,3]`、参考相关 0.650）
- RVC 验证：`tensor_count=457`、`info=alxy`、`sr=48k`、`unexpected=[]`、`missing=103` 全部 `enc_q.*`、`infer_shape=(1,1,19200)`、`infer_finite=True`、`RESULT=PASS`。

## 3. 复现命令

~~~powershell
& 'F:/ai歌曲/RVC20260311/runtime/python.exe' `
  'F:/Deepseek_harness/AI音乐处理/08_项目代码/ams-tools/msvc2pth/msvc2pth.py' `
  'F:/ai配音/心月-变声器/心月变声/_internal/modules/爱莉希雅/爱莉希雅.msvc' `
  --out 'F:/Deepseek_harness/AI音乐处理/07_执行计划/验证/msvc2pth_out' `
  --reference-pth 'F:/ai歌曲/0 爱莉希雅/爱莉希雅.pth'

& 'F:/ai歌曲/RVC20260311/runtime/python.exe' `
  'F:/Deepseek_harness/AI音乐处理/08_项目代码/ams-tools/msvc2pth/verify_rvc.py' `
  'F:/Deepseek_harness/AI音乐处理/07_执行计划/验证/msvc2pth_out/爱莉希雅/爱莉希雅.pth' `
  'F:/ai歌曲/RVC20260311'
~~~

## 4. 40k 模型剩余问题

- 张量表可完整恢复并通过键集校验（与 40k 参考 manbo 的 457 keys 完全一致）。
- 失败点在**数据分片边界**：可打印 run 的两端混入 1–2 个数据字节，40k 两个文件的尾部 padding/边界与 48k 不同，当前固定规则导致分片长度差 1–6 字节，进而 offsets 错位。
- 方案：把边界校准做成小范围搜索（每个边界 ±0–8，以参考模型 log-std 相关 + 有限值比例为目标），并缓存每文件结果；预计 1 人日内完成（M0.5）。

## 5. 产物与安全

- 产物在 `07_执行计划/验证/msvc2pth_out/`；只保留已验证的爱莉希雅，WIP 输出已清理。
- 不生成 index；不提供 `.cyd` 解密；不修改源目录；仅本机用户自有文件互操作。
