# TTS 音源炼制插件最终状态（2026-10-26）

## 结果

Phase 1-3 完成；全量 640 条闲云 TTS 音源已完成 20 epoch 续训、导出、注册并通过 Gateway 合成与评测。

## 四个问题的处理

### 1. 只跑 1 epoch

已从全量 1 epoch checkpoint 继续训练 20 epoch：

    final checkpoint:
    F:/AMS-Studio/data/tts_train/xianyun_final_run2/lightning_logs/version_0/checkpoints/epoch=3289-step=2466546.ckpt

最终音源：

    F:/AMS-Studio/data/tts_voices/闲云/
      闲云.onnx        63,511,037 bytes
      闲云.onnx.json        7,732 bytes
      voice.json

### 2. clipping

TTS Gateway 已加入峰值 limiter：

- 目标 0.98（约 -0.17 dBFS）；
- 仅超过阈值时缩放；
- 最终音源测试 clipping=0，peak=0.97995。

测试：

    job_d6644a0494b64861 succeeded
    job_f6df95a05a654383 succeeded
    16.1663s / 22050Hz / clipping 0 / peak 0.97995

### 3. 音素化 shim 与 num_workers

- Windows 没有 piper-phonemize wheel，继续使用 espeak-ng.dll shim；
- 补齐 @ S A N E X I [ 音素；
- 测试 num_workers=2 会卡住 DataLoader，最终稳定使用 num_workers=0；
- 独立 runtime + batch 4 完成 20 epoch；
- 这是当前 Windows 原生路径的已知性能限制，不是训练错误。

### 4. 独立 runtime

已复制独立 Python：

    F:/AMS-Studio/data/runtime/tts-train/0.1.0/python/
      python.exe
      Lib/site-packages/...
      torch 2.0.0+cu118

正式包：

    F:/Deepseek_harness/AI音乐处理/09_发布/_TTS_TRAIN_RUNTIME/
      runtime.tts-train-0.1.0.zip  6,952,733,515 bytes
      part-001 ... part-004
      SHA256SUMS-parts.txt

runtime.json / MANIFEST.json 已生成。

## 训练与模型

数据集：

    F:/AMS-Studio/data/datasets/tts/xianyun_full
    640 wav / 393,066,206 bytes

最终模型：

    F:/AMS-Studio/data/tts_voices/闲云/闲云.onnx

哈希：

    闲云.onnx
    67F2F2F0C20F7AF629C3BA68BD5690B525A3C83FB889B6958295DEE20E43C324

    闲云.onnx.json
    DED36CEBAEACEA9D502DB94D10ACC391090C9A355932FBFDDB7F4433ACF0D6CB

## 上传分工

GitHub：

- 源码 6 个仓库；
- H2 0.2.1 现有 release 不动；
- H3 setup/feed 2046/manifest/SHA/gateway/plugin/闲云 ONNX/发布说明。

Hugging Face：

- runtime.tts-train 6.95GB 四个分片；
- huayan 基础 checkpoint 845,898,456 bytes；
- 现有 runtime.rvc/runtime.tts/kernel/voicepack；
- 可选最终 TTS 模型归档。

详细清单见 H2_H3_REVALIDATION.md。

## 当前服务

Gateway 端口以 data/runtime/ports.json 为准；最后为 14624。
H2 latest 仍为 feed 2044 / setup 0.2.1。
H3 RC draft 未发布，需以 TTS 最终状态重建 feed 2046。
