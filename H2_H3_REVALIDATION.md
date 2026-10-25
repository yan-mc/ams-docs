# H2/H3 重新校验与上传分工（2026-10-26 最终）

## 1. 当前状态

### H2

- 状态：完成，保持 0.2.1 / feed 2044 发布线，不重发。
- 闲云 RVC：F:/AMS-Studio/data/voices/闲云/闲云.pth + 闲云.index。
- TTS 默认与闲云均已验证。
- TTS 插件新增功能不改变 H2 latest。

### TTS 插件

- Phase 1-3 完成。
- 全量 640 条闲云数据集。
- 最终 20 epoch 训练：epoch=3289。
- 最终音源：F:/AMS-Studio/data/tts_voices/闲云/。
- 削波 limiter：clipping 0。
- 独立 tts-train runtime：6.95GB，已分 4 片。
- num_workers=2 在 Windows 会卡住，稳定使用 0；独立 runtime + batch 4 已可用。

### H3

- 状态：draft v1.0.0-rc.1，未发布。
- 原 RC 发布物早于 TTS 插件，已过期。
- 需要重新生成 feed 2046 和 RC 资产。
- H3 签收仍等 H2 最终关闭，但不能把旧 RC 当最终。

## 2. H2 重校验步骤

1. 确认 H2 latest：feed 2044 / setup 0.2.1；
2. 回归 RVC、pymss、TTS、评测、混音；
3. 验证闲云 TTS；
4. 验证 limiter（peak <= 0.98，clipping=0）；
5. 确认 H2 release 资产不替换；
6. 更新 H2_RESULTS，只记录 H2。

## 3. H3 重校验步骤

1. 以最终 TTS 音源和 runtime 为基线；
2. 重建 Gateway/Plugin；
3. 更新安装器（launcher 不变则复用 1.0.0-rc.1）；
4. 生成 feed 2046：
   - gateway 1.0.0-rc.1
   - plugin 1.0.0-rc.1
   - runtime.tts-train 0.1.0
   - setup 1.0.0-rc.1
5. 更新 manifest/SHA/签名；
6. 干净目录安装、升级、回滚；
7. UI CDP 12 页签 + TTS 数据集 + TTS 炼制；
8. 归档旧 GitHub 资产；
9. 发布 RC/最终 H3。

## 4. 上传 GitHub

仓库：yan-mc/ams-release

### H2

- 已存在 v0.2.1 release：不重复上传；
- 源码/文档走 GitHub 仓库。

### H3

必须：

- AMS-Studio-Setup-1.0.0-rc.1.exe
- ams-components.json（feed 2046，签名）
- RELEASE_MANIFEST.json
- SHA256SUMS.txt
- service-gateway-1.0.0-rc.1.zip
- plugin-ams-music-bundle-1.0.0-rc.1.zip
- 闲云.onnx（63,511,037 bytes）
- 闲云.onnx.json（7,732 bytes）
- 发布说明/兼容矩阵

源码：

- ams-launcher
- ams-plugins
- ams-services
- ams-release
- ams-docs
- AMS-Studio

GitHub 单资产限制 2GB，所以 runtime 不上 GitHub。

## 5. 上传 Hugging Face

数据集仓库：yan2002/AI-MUSIC-STUDIO

必须：

- runtime.tts-train/0.1.0 分片：
  - part-001 1,900,000,000 bytes
  - part-002 1,900,000,000 bytes
  - part-003 1,900,000,000 bytes
  - part-004 1,252,733,515 bytes
  - 总 zip 6,952,733,515 bytes
- 基础 checkpoint：
  epoch=3269-step=2460540.ckpt（845,898,456 bytes）
- 现有 runtime.rvc / runtime.tts / kernel / voicepack
- 可选 TTS 最终模型归档（若同时放 HF）

不要上传到 GitHub：

- tts-train runtime（6.95GB）
- RVC runtime
- kernel / voicepack
- 大型数据集

## 6. 最终音源哈希

F:/AMS-Studio/data/tts_voices/闲云/闲云.onnx

SHA256：

1653D57C4C8CCE42739093F6A9BC8069DF4D0D910AAD4DDCE8F7E30A973597EB

闲云.onnx.json：

DED36CEBAEACEA9D502DB94D10ACC391090C9A355932FBFDDB7F4433ACF0D6CB

runtime zip：

CFB75117292A92963715C77EC7AA9208A27761A1039E4348BC88D1B7DB2AB9AC

分片哈希在：

F:/Deepseek_harness/AI音乐处理/09_发布/_TTS_TRAIN_RUNTIME/SHA256SUMS-parts.txt

## 7. 最终验收

- 闲云 TTS：job_d6644a0494b64861 succeeded；
- 评测：job_f6df95a05a654383 succeeded；
- 16.1663s / 22050Hz / clipping 0；
- Peak 0.97995；
- 模型列表可见“闲云”。

## 8. 结论

H2 保持完成状态；H3 需要以 TTS 最终音源和 feed 2046 重建 RC。GitHub 放小发布物/源码，Hugging Face 放 6.95GB 训练 runtime、checkpoint 和大资产。
