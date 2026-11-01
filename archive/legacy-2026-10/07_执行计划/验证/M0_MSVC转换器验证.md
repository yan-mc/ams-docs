# M0 验证：MSVC → RVC v2 .pth 转换器（M0.5 最终：3/3 PASS，40k 音质收尾完成）

> 时间：2026-10-19｜任务：task-18 / task-24（xinyue-analyst）｜对象：心月-变声器 `_internal/modules/*/*.msvc`（本机 3 个）
> 工具：`08_项目代码/ams-tools/msvc2pth/`（v0.4.0）｜验证环境：`F:/ai歌曲/RVC20260311/runtime/python.exe`（Python 3.9.13 + torch 2.0.0，CPU）

## 1. 结论（3/3 PASS）

| 模型 | 容器/张量表 | 457 张量键集 | pth | RVC load | RVC infer（幅度） | 脏值 | 结果 |
|---|---|---|---|---|---|---|---|
| 爱莉希雅（48k） | ✅ | ✅ | 57,580,765 B | unexpected=0，missing 103 全 enc_q | ✅ -0.262 ~ 0.569 | 1（footer 残留，无害） | **PASS** |
| 法国赌神（40k） | ✅ | ✅ | 55,221,469 B | unexpected=0，missing 103 全 enc_q | ✅ **-0.308 ~ 0.359（不饱和）** | **0** | **PASS** |
| 曼波Pro（48k） | ✅ | ✅ | 57,580,765 B | unexpected=0，missing 103 全 enc_q | ✅ -0.475 ~ 0.836 | 1（footer 残留，无害） | **PASS** |

**结论**：MSVC 是容器混淆（14B 头 + 3 条反转 TLV + 16B trailer），并非加密/改架构：metadata + 张量表 JSON + 明文 FP16 数据分片重排。3 个模型均还原为标准 RVC v2 `.pth` 并被 RVC 运行时加载+推理（`verify_rvc.py` 全部 exit=0）。

## 2. 产物

```
07_执行计划/验证/
├─ M0_MSVC转换器验证.md              # 本文件
└─ msvc2pth_out/
   ├─ calibration_cache.json         # 3 文件校准缓存副本（shard_ranges/perm/mask）
   ├─ 爱莉希雅/{爱莉希雅.pth, conversion.json, verify.txt}
   ├─ 法国赌神/{法国赌神.pth, conversion.json, verify.txt}
   └─ 曼波Pro/{曼波Pro.pth, conversion.json, verify.txt}
```

## 3. 校准参数（最终）

| 模型 | 源 sha256 前 16 | 分片长度（物理序，逻辑序 (2,4,0,1,3)） | 参考相关 | mask | max_end |
|---|---|---|---|---|---|
| 爱莉希雅 | 308d84a9500875ad | 2508, 997, 57428987, 999, 497 | 0.650 | — | 57,433,988 |
| 法国赌神 | 79e631fe3e71f9fd | 2508, 994, 55069690, 1000, 500 | 0.9986 | [55072690, 8] | 55,074,692 |
| 曼波Pro | bf0b63b10559c848 | 2502, 1000, 57428986, 1000, 500 | 0.657 | — | 57,433,988 |

硬约束：**各分片长度偶数（FP16 对齐）+ 总长 == 张量表 max_end**；评分：参考模型逐张量 log-std 相关 + fp16 坏值数。

## 4. 法国赌神（40k）音质收尾（task-24）

### 4.1 根因（证据）

| 现象 | 证据 |
|---|---|
| 201 个脏值集中在 `enc_p.proj.weight` 尾部 1,234 字节 | 组装后按张量定位：201 个非有限/\|v\|>20 值全部落在 offset 54,926,468+；映射回物理位置 = gap1 `[4001,4995)`（200 个）+ gap0 尾部 8 字节 footer（1 个） |
| gap1 边界错位 1 字节 | 把 gap1 按 fp16 解释：`[4001,4995)` bad=200、max=5.87e4；改为 `[4000,4994)` 后 bad=**0**、max=0.178（起点应包含 run1 末字节 `0x40`，终点前移 1） |
| big 分片长度奇偶 | raw big 长度 55,069,689（奇）→ 后续段整体 FP16 错位；取 `[50876,55120566)` 长度 55,069,690（偶）后 bad=0 |
| 剩余 1 个值 | 8 字节 footer `50 52 56 31 b4 c6 00 00`（fp16 = 50.5 / 0.17 / **-6.70** / 0.0），容器计入数据长度；容器内无替代数据字节（run3 lead `*n3`、run4 trail `ype":` 实测均为文本）→ 用 `mask_data_ranges` 显式归零 |

### 4.2 修复前后对比

| 指标 | 修复前（task-18） | 修复后（task-24） |
|---|---|---|
| 分片长度 | 2508, 994, 55069688, 1000, 502 | 2508, 994, **55069690**, 1000, **500** |
| 脏值/哨兵 | 201（全部归零） | **0**（footer 8 字节显式 mask） |
| `enc_p.proj.weight` std | 14.25（参考 0.0677） | **0.06739**（参考 0.06772） |
| `enc_p.proj.weight` \|max\| | 976 | **1.646**（参考 1.63） |
| `enc_p.proj.bias` std / max | — / — | 0.07250 / 0.348（参考 0.07224 / 0.355） |
| RVC infer 输出 | 0.316 ~ 1.0（饱和） | **-0.308 ~ 0.359（正常）** |
| 与 manbo 逐张量 std 偏差 dlog | 最大 5.35 | **中位 0.005 / 均值 0.012 / 最大 0.337** |

逐张量对比（40k 新 pth vs `manbo.pth`）：除 3 个极小权重（dlog 0.23~0.34）外，其余 454 个张量 log-std 偏差 < 0.12；整体 \|max\|=6.613（参考 6.637）。两模型逐值完全相等仅 0.18%（同谱系、非同一权重），但统计分布高度一致。

## 5. 复现命令

~~~powershell
$py = 'F:/ai歌曲/RVC20260311/runtime/python.exe'
$tool = 'F:/Deepseek_harness/AI音乐处理/08_项目代码/ams-tools/msvc2pth'
$out = 'F:/Deepseek_harness/AI音乐处理/07_执行计划/验证/msvc2pth_out'

& $py "$tool/msvc2pth.py" 'F:/ai配音/心月-变声器/心月变声/_internal/modules/爱莉希雅/爱莉希雅.msvc' `
    --out $out --reference-pth 'F:/ai歌曲/RVC20260311/assets/weights/爱莉希雅.pth' --name 爱莉希雅
& $py "$tool/msvc2pth.py" 'F:/ai配音/心月-变声器/心月变声/_internal/modules/法国赌神/法国赌神.msvc' `
    --out $out --reference-pth 'F:/ai歌曲/RVC20260311/assets/weights/manbo.pth' --name 法国赌神
& $py "$tool/msvc2pth.py" 'F:/ai配音/心月-变声器/心月变声/_internal/modules/曼波Pro/苏苏.msvc' `
    --out $out --reference-pth 'F:/ai歌曲/RVC20260311/assets/weights/爱莉希雅.pth' --name 曼波Pro

# 验证（3 次，期望 RESULT=PASS / exit=0）
foreach ($n in '爱莉希雅','法国赌神','曼波Pro') {
  & $py "$tool/verify_rvc.py" "$out/$n/$n.pth" 'F:/ai歌曲/RVC20260311'
}

# 分片边界诊断（新文件）
& $py "$tool/msvc2pth.py" 'F:/ai配音/心月-变声器/心月变声/_internal/modules/法国赌神/法国赌神.msvc' --diagnose
~~~

## 6. 剩余风险

1. 48k 两模型仍各有 1 个 footer 残留值被归零（爱莉希雅 `emb_g.weight`、曼波Pro `enc_p.proj.weight`），infer 幅度正常、无饱和；可用同样的 `mask_data_ranges` 清掉（低优先级，未纳入本次验收）。
2. 法国赌神 infer 幅度（std≈0.051）略小于 manbo 参考（std≈0.094）：权重统计已高度一致，差异可能来自模型本身微调；最终音色建议用真实音源试听确认。
3. 边界校准依赖参考模型；新变体需先用 `--diagnose` 检查并人工确认后写入缓存。
4. 不生成 index；不提供 .cyd 解密；不修改源目录。

## 7. 产物与安全

- 仅处理本机用户合法持有的 .msvc 文件；输出仅在验证目录。
- 未生成 .index（MSVC 无检索文件）；未解密 .cyd；未修改 `F:/ai配音`、`F:/ai歌曲` 源目录。
