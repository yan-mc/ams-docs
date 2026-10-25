# H2/H3 最终校验与上传状态（2026-10-26）

## TTS 对齐修复

- 根因 1：缺少 Piper pad intersperse；
- 根因 2：旧 shim 使用 espeak_TextToPhonemes(0x100)，官方使用 TextToPhonemesWithTerminator(0x02)，并按 terminator 补标点/空格、NFD；
- 修复后重训 v3 20 epoch；
- ASR 对照：官方 huayan 0.833，闲云 v3 0.833；
- 最终音源：F:/AMS-Studio/data/tts_voices/闲云/；
- ONNX SHA256：67F2F2F0C20F7AF629C3BA68BD5690B525A3C83FB889B6958295DEE20E43C324；
- Gateway job_2eae4d3afedc40f1 succeeded，clipping 0，peak 0.97995。

## H2 GitHub 清理

v0.2.1 已从 62 个资产清理为 10 个实际资产：

- ams-components.json
- AMS-Studio-Setup-0.2.1.exe
- RELEASE_MANIFEST.json
- SHA256SUMS.txt
- ams-kernel-installlocal-0.2.1-rc.2-final.zip
- catalog-models-2026.10.19.zip
- voicepack-manbo.zip（唯一 RVC 音源）
- service-gateway-0.1.0-stageg2.zip
- plugin-ams-music-bundle-0.3.0-stageg2.zip
- service-vc-0.1.0-mix.zip

已删除：

- 所有 stage/test plugin/gateway 包
- runtime-tts-0.1.0-part-001（GitHub 上未使用；feed 指向 HF）
- tool-msvc GitHub 资产
- 旧 0.2.0 MSI

已修复 SHA256SUMS 中不存在的 ams-components-v44.json 引用。

H2 latest 仍为 feed 2044 / setup 0.2.1。

## H3 GitHub

- 已删除过期 draft v1.0.0-rc.1（含旧 feed/二进制）。
- 当前 GitHub 只剩 v0.2.1 / v0.2.0。
- H3 未发布；需要用最终 TTS/feed 2046 重新生成。

## GitHub 上传规则

只上传：

- 发布安装器、feed、manifest、SHA
- H2 当前 gateway/plugin/VC 资产
- 曼波 RVC voicepack（唯一音源）
- 源码仓库
- 发布说明

不上传：

- TTS 音源（tts_voices/闲云）
- 测试音源
- 测试日志
- 中间计划
- 6.95GB runtime

## Hugging Face

应上传：

- runtime.tts-train 0.1.0 四片
- huayan 基础 checkpoint
- 现有 runtime.rvc/runtime.tts/kernel/voicepack
- 不传 TTS 音源

当前状态：

- 本地已生成 runtime.zip 6,952,737,876 bytes，SHA256 9975B7E3B3433122A20CC0D2359BE27D772683146C860DFBFC77B1B782FDE2F4；
- 四片和 component.json 已生成；
- Hugging Face 凭据无效（whoami 返回 Invalid username or password），上传被阻塞；
- 旧 HF runtime.tts-train 尚未上传（不覆盖旧 runtime.tts）。

## 当前发布状态

- H2：完成，GitHub 已清理。
- H3：未发布，旧 draft 已删除。
- TTS：v3 音源完成，本地验收通过；GitHub/HF 均未上传。
- 下一步：提供有效 HF token，上传 runtime.tts-train；然后重建 H3 feed 2046。
