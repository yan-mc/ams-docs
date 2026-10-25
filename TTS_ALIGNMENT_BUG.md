# TTS 文字/声音不对齐 Bug 报告（2026-10-26）

## 现象

用 tts_voices/闲云 合成时，声音和文字明显不对应，疑似模型输入/训练序列错位。

## 根因

自定义 piper_phonemize shim 的 phoneme_ids_espeak 少做了 Piper 官方的 pad interspersing。

官方 phoneme_ids.cpp：

- PhonemeIdConfig.interspersePad = true
- BOS 后插入 pad
- 每个 phoneme 后插入 pad
- 最后 EOS

错误实现：

    BOS + phoneme ids + EOS

正确实现：

    BOS + PAD + (phoneme + PAD)* + EOS

## 证据

修复前 dataset.jsonl 首条：

    [1, 32, 31, 5, 0, 132, ...]

修复后：

    [1, 0, 32, 0, 31, 0, 5, 0, 152, 0, 132, ...]

旧训练数据没有 pad，Piper 推理时会自动 intersperse pad，训练/推理分布不一致，所以合成内容错乱。

## 修复

- shims/piper_phonemize.py 已按官方默认值插入 pad；
- 640 条 xianyun_full 重新 preprocess：
  F:/AMS-Studio/data/tts_train/xianyun_full_v2/preprocessed
- 从 huayan medium epoch=3269 重新 fine-tune 20 epoch；
- 旧 tts_voices/闲云 标记为无效，训练完成后替换。

## 影响

- 旧 20 epoch 闲云 TTS 不可作为验收音源；
- H2 的 TTS 功能测试仍有效（合成链路）；
- H2 验收不能使用旧闲云 TTS 音质结论；
- H3 发布前必须使用修正后的音源；
- runtime 包需要重新打包（旧 zip 含错误 shim）。

## 待完成

- v2 20 epoch 完训；
- export ONNX；
- 替换 tts_voices/闲云；
- Gateway 合成/评测；
- 重新打包 runtime；
- 更新 H2/H3 验收记录。


## 修复完成（2026-10-26 01:06）

- 修正 shim 后重新 preprocess 640 条：
  F:/AMS-Studio/data/tts_train/xianyun_full_v2/preprocessed
- 从 huayan medium epoch=3269 重新 fine-tune 20 epoch：
  epoch=3289-step=2466260.ckpt
- 新音源：
  F:/AMS-Studio/data/tts_voices/闲云/闲云.onnx
  SHA256 67F2F2F0C20F7AF629C3BA68BD5690B525A3C83FB889B6958295DEE20E43C324
- 旧错误音源已备份：
  F:/AMS-Studio/data/backups/tts_闲云_wrong_pad
- Gateway 合成：job_f80005e750354d34 succeeded
- 评测：job_104acca9ec04435e succeeded，clipping 0，peak 0.97995
