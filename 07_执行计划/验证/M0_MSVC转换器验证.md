# M0 验证：MSVC → RVC v2 .pth 转换器（M0.5 更新：3/3 通过）

> 时间：2026-10-19｜任务：task-18｜对象：心月-变声器 `_internal/modules/*/*.msvc`（本机 3 个）
> 工具：`08_项目代码/ams-tools/msvc2pth/`（v0.3.0）｜验证环境：`F:/ai歌曲/RVC20260311/runtime/python.exe`（Python 3.9.13 + torch 2.0.0，CPU）

## 1. 结论

| 模型 | 容器/张量表 | 457 张量键集 | 生成 pth | RVC load | RVC infer | 结果 |
|---|---|---|---|---|---|---|
| 爱莉希雅（48k） | ✅ | ✅ | ✅ 57,580,765 B | unexpected=0，missing 103 全 enc_q | ✅ 有限（-0.262 ~ 0.569） | **PASS** |
| 法国赌神（40k） | ✅ | ✅ | ✅ 55,221,469 B | unexpected=0，missing 103 全 enc_q | ✅ 有限（0.316 ~ 1.0） | **PASS**（保真度见 §5） |
| 曼波Pro（48k） | ✅ | ✅ | ✅ 57,580,765 B | unexpected=0，missing 103 全 enc_q | ✅ 有限（-0.475 ~ 0.836） | **PASS** |

**结论**：MSVC 不是改架构/加密权重，只是容器混淆（14B 头 + 3 条反转 TLV + 16B trailer；metadata + 张量表 JSON + 明文 FP16 + 分片重排）；3 个模型均可还原为标准 RVC v2 `.pth`，被 RVC 运行时加载并推理（`verify_rvc.py` 均 exit=0）。

## 2. 产物

```
07_执行计划/验证/
├─ M0_MSVC转换器验证.md              # 本文件
└─ msvc2pth_out/
   ├─ calibration_cache.json         # 3 个文件的校准参数缓存副本
   ├─ 爱莉希雅/{爱莉希雅.pth, conversion.json, verify.txt}
   ├─ 法国赌神/{法国赌神.pth, conversion.json, verify.txt}
   └─ 曼波Pro/{曼波Pro.pth, conversion.json, verify.txt}
```

各文件 `conversion.json` 记录：源/输出 sha256、metadata、JSON 片段顺序/裁剪、**校准参数（shard_lengths/shard_order/max_end/参考相关/哨兵归零明细）**、warnings。

## 3. 校准参数（M0.5 核心结果）

| 模型 | 源 sha256 前 16 位 | 源大小 | 分片长度（物理序） | 逻辑顺序 | 参考相关 log-std | 哨兵归零 | max_end |
|---|---|---|---|---|---|---|---|
| 爱莉希雅 | 308d84a9500875ad | 57,748,698 | 2508, 997, 57428987, 999, 497 | (2,4,0,1,3) | 0.650 | 1 | 57,433,988 |
| 法国赌神 | 79e631fe3e71f9fd | 55,389,446 | 2508, 994, 55069688, 1000, 502 | (2,4,0,1,3) | 0.9995 | 201 | 55,074,692 |
| 曼波Pro | bf0b63b10559c848 | 57,748,717 | 2502, 1000, 57428986, 1000, 500 | (2,4,0,1,3) | 0.657 | 1 | 57,433,988 |

- 48k 两模型布局与 M0 已验证的 `(2,4,0,1,3)` 一致（`used_layout_prior=true`）。
- 40k 与 48k 的主要差异：大分片长度/尾部小分片长度不同（JSON 片段尾部换行/可打印污染字节不同），通过“分片总长 == max_end + 全偶长度 + 参考 log-std 相关”网格搜索确定；曼波Pro 需把 shard0 缩短 6 字节（尾部无换行导致）。
- 校准结果按「源 sha256 + 参考 sha256」缓存于工具 `calibration_cache.json`（`shard_ranges` 为 rev 内绝对区间），重复转换直接命中缓存。

## 4. 复现命令

~~~powershell
$py = 'F:/ai歌曲/RVC20260311/runtime/python.exe'
$tool = 'F:/Deepseek_harness/AI音乐处理/08_项目代码/ams-tools/msvc2pth'
$out = 'F:/Deepseek_harness/AI音乐处理/07_执行计划/验证/msvc2pth_out'

# 爱莉希雅（48k，参考 48k）
& $py "$tool/msvc2pth.py" 'F:/ai配音/心月-变声器/心月变声/_internal/modules/爱莉希雅/爱莉希雅.msvc' `
    --out $out --reference-pth 'F:/ai歌曲/RVC20260311/assets/weights/爱莉希雅.pth' --name 爱莉希雅
# 法国赌神（40k，参考 40k manbo）
& $py "$tool/msvc2pth.py" 'F:/ai配音/心月-变声器/心月变声/_internal/modules/法国赌神/法国赌神.msvc' `
    --out $out --reference-pth 'F:/ai歌曲/RVC20260311/assets/weights/manbo.pth' --name 法国赌神
# 曼波Pro（48k，参考 48k）
& $py "$tool/msvc2pth.py" 'F:/ai配音/心月-变声器/心月变声/_internal/modules/曼波Pro/苏苏.msvc' `
    --out $out --reference-pth 'F:/ai歌曲/RVC20260311/assets/weights/爱莉希雅.pth' --name 曼波Pro

# 验证（3 次，期望 RESULT=PASS / exit=0）
foreach ($n in '爱莉希雅','法国赌神','曼波Pro') {
  & $py "$tool/verify_rvc.py" "$out/$n/$n.pth" 'F:/ai歌曲/RVC20260311'
}
~~~

## 5. 已知限制（务必阅读）

1. **法国赌神保真度低于另两个模型**：数据分片边界仍有少量无法定位的容器哨兵字节（NaN/Inf 或 |v|>20，共 201 个值被归零），infer 输出有限但偏饱和（0.316~1.0）。验收标准（load+infer PASS）已满足；作为最终音色使用前建议在 RVC 运行时内做中间张量对比/动态调试，进一步收窄尾部边界。
2. 分片边界定位部分依赖参考模型（48k 用爱莉希雅标准 pth、40k 用 manbo 标准 pth）；无参考时仅按 fp16 合理性评分，置信度低。
3. 张量碎片拼接依赖 5 段固定形态，新变体工具会明确报错（不猜测）。
4. 不生成 index；不提供 .cyd 解密；不修改源目录。

## 6. 产物与安全

- 仅处理本机用户合法持有的 .msvc 文件；输出仅在验证目录。
- 未生成 .index（MSVC 无检索文件）；未解密 .cyd；未修改 `F:/ai配音`、`F:/ai歌曲` 源目录。
