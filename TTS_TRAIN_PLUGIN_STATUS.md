# TTS 音源炼制插件 Phase 1-3 完成记录（2026-10-26）

## 结论

TTS 音源炼制插件已完成 Phase 1-3 的可运行闭环，并用 640 条完整闲云数据集完成训练、ONNX 注册和 TTS 合成测试。

## Phase 1：运行时与基础模型

- Windows 原生 Piper/VITS 训练链路
- espeak-ng.dll + 自定义 piper_phonemize shim
- numba monotonic_align fallback
- 补齐缺失 IPA 音素：@ S A N E X I [
- 中文 huayan medium checkpoint：
  F:/AMS-Studio/data/runtime/tts-train/0.1.0/checkpoints/epoch=3269-step=2460540.ckpt
- 1 epoch fine-tune 成功，ONNX 导出并由 Piper 加载

## Phase 2：Gateway

新增 job 类型：

- train.tts
- tts.dataset.prepare
- 复用 tts.synthesize

train.tts 流程：

1. 校验 metadata.csv + wav/
2. piper_train.preprocess
3. medium checkpoint fine-tune
4. 自动发现 checkpoint
5. export_onnx
6. 复制 .onnx.json
7. 原子注册 data/tts_voices/<name>
8. artifacts: model/config

已通过：

- job_407162acfa0d4800 train.tts succeeded
- job_dc34b618661a458a tts.dataset.prepare succeeded

## Phase 3：UI

已增加页签：

- TTS 数据集
- TTS 音源炼制

UI 支持：

- 数据集目录/名称
- 语言 cmn/en-us
- epochs / batch
- base checkpoint
- quality
- 任务提交/进度/取消（通用 Jobs 面板）
- 训练完成后模型进入 TTS 模型列表

UI 包已构建并部署到：

F:/AMS-Studio/data/kernel/dsh/profiles/desktop/node_modules/@ams/plugin-ui

刷新 AMS 页面即可看到新页签。

## 完整闲云数据集测试

数据集：

    F:/AMS-Studio/data/datasets/tts/xianyun_full
    640 wav
    393,066,206 bytes
    metadata.csv

训练：

    job_c43295f1661d4534
    status: succeeded
    progress: 100.0
    stage: done
    model: xianyun_tts_full
    epochs: 1
    base: huayan medium epoch=3269

模型：

    F:/AMS-Studio/data/tts_voices/xianyun_tts_full/
      xianyun_tts_full.onnx       63,511,037 bytes
      xianyun_tts_full.onnx.json       7,732 bytes
      voice.json

合成：

    job_39a55eeb39d44ea1 succeeded
    text: 你好，我是闲云。这是使用完整数据集炼制的语音。
    duration: 7.8652s
    sample_rate: 22050

评测：

    job_0365e2e073614bcc succeeded
    peak: 0.999969
    clipping_samples: 2
    rms_dbfs: -14.64
    lufs: -13.5
    silence_ratio: 0.139

## 注意

- 这是 1 epoch fine-tune，主要用于全量链路验收，不是最终音质版本。
- 输出有 2 个 clipping samples，后续可在合成后加安全限幅。
- 运行时依赖目前复用 RVC Python + data/runtime/tts-train 的 deps；正式发布需要打包成独立 runtime 组件。
- 当前 Gateway 端口以 data/runtime/ports.json 为准（最后为 10198）。
- 未继续 H3。

## 关键路径

- 训练脚本：ams-services/gateway/app/runners/scripts/tts_train.py
- 数据集脚本：ams-services/gateway/app/runners/scripts/tts_dataset.py
- Runner：ams-services/gateway/app/runners/tts_train.py
- UI：ams-plugins/packages/plugin-ui/src/client.js
