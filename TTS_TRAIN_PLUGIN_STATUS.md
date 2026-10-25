# TTS 音源炼制插件 Phase 0 状态（2026-10-25）

## 结论

Windows 原生 Piper VITS 训练链路已打通；不需要 WSL、不需要 C 盘 AMS data。

当前只完成 PoC，不是完整插件。

## 已打通

### 1. 训练运行时

目录：

F:/AMS-Studio/data/runtime/tts-train/0.1.0

包含：

- Piper 官方训练源码
- espeak-ng.dll + espeak-ng-data
- 自定义 piper_phonemize shim
- numba monotonic_align fallback
- pytorch-lightning 1.9.5 deps
- 复用 RVC 运行时的 Python 3.9 + torch 2.0 + CUDA

### 2. 音素化

- 无 Windows piper-phonemize wheel；
- 使用现有 espeak-ng.dll；
- 通过 ctypes 调用 espeak_TextToPhonemes；
- 生成中文 IPA 音素；
- 适配 piper_train.preprocess。

### 3. 预处理

20 条闲云片段：

    data/datasets/tts/xianyun_smoke
      wav/
      metadata.csv

输出：

    data/tts_train/xianyun_smoke_pre
      config.json
      dataset.jsonl
      cache/

结果：成功。

### 4. 训练

修正：

- num_workers 默认 1 -> 0（Windows DataLoader spawn 问题）；
- VAD 显式 CPUExecutionProvider；
- phoneme_ids 必须是扁平 int 列表。

结果：

    job 本地 1 epoch
    TRAIN_OK
    checkpoint: data/tts_train/xianyun_smoke_run/lightning_logs/version_5/checkpoints/epoch=0-step=14.ckpt

### 5. ONNX 导出

    data/tts_train/xianyun_smoke.onnx
    63,511,037 bytes

Piper 加载成功，输出：

    data/tts_train/xianyun_smoke.wav
    36,000 bytes

## 当前已知问题

1. 音素表只有 152 项，espeak 输出缺 8 个符号：
   @ S A N E X I [
   需要完整 espeak map，否则部分中文会退化为 padding。

2. 当前是从零训练 1 epoch，只验证链路，不是可用音质。

3. 还没有下载/接入 medium 基础 checkpoint。

4. 还没有 Gateway train.tts job。

5. 还没有 UI 数据集/训练页。

6. 还没有把训练 runner 打包成正式 runtime 组件。

## 下一阶段

Phase 1：

1. 生成完整 espeak 音素表；
2. 下载中文/通用 medium checkpoint 到 data；
3. 1 epoch fine-tune 验证；
4. 输出可听的测试音频。

Phase 2：

1. Gateway train.tts / tts.dataset.prepare / tts.export；
2. 进度、取消、恢复；
3. 自动注册到 data/tts_voices。

Phase 3：

1. UI TTS 音源炼制页；
2. 数据集导入/校验；
3. 训练参数/模型试听。

## 注意

- 当前所有 PoC 数据和依赖都在 F:/AMS-Studio/data。
- 没有写 C 盘 AMS data。
- 没有启动正式 TTS 训练。
- H2/H3 未继续。
