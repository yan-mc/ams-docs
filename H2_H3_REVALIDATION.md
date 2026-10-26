# H2/H3 完成状态与上传清单（2026-10-26）

## H2

- 状态：完成；
- latest：v0.2.1 / feed 2044 / setup 0.2.1；
- GitHub v0.2.1 资产：10 个；
- 唯一 RVC 音源：voicepack-manbo.zip；
- 无 TTS 音源；
- SHA256SUMS 已修正（ams-components.json）；
- H2 不重发。

## H3

- 状态：已发布 prerelease v1.0.0-rc.1；
- latest 仍是 H2 v0.2.1；
- feed：2046；
- setup：1.0.0-rc.1；
- Gateway：1.0.0-rc.1；
- Plugin：1.0.0-rc.1；
- runtime.tts-train：0.1.0；
- GitHub 资产 6 个：
  - ams-components.json
  - AMS-Studio-Setup-1.0.0-rc.1.exe
  - service-gateway-1.0.0-rc.1.zip
  - plugin-ams-music-bundle-1.0.0-rc.1.zip
  - RELEASE_MANIFEST.json
  - SHA256SUMS.txt
- 无 TTS 音源。

## Hugging Face

已上传：

- runtime-packages/runtime.tts-train/0.1.0/component.json
- runtime.tts-train-0.1.0.zip.part-001
- part-002
- part-003
- part-004

四个分片 LFS oid 与 component SHA256 全部一致：

- e74d2721...
- d0d1eb6b...
- 29134ac6...
- a5b98230...

TTS 音源 闲云 留在本机，不上传 GitHub/HF。

## TTS

- 训练模式冻结；
- 最终 v3 音源：F:/AMS-Studio/data/tts_voices/闲云/；
- 念字正确；断句/听感仍有问题，按用户要求暂不继续改；
- ASR：官方 huayan 0.833 / 闲云 v3 0.833；
- 不进入 H2/H3 上传。

## 干净目录

- H3 setup 1.0.0-rc.1 安装成功；
- 卸载保留 data marker；
- H3 测试目录已清理。

## 下一步

- H4 验收；
- H3 UI CDP 12 页签 + TTS 数据集/炼制页；
- 旧资产归档；
- 最终 H5 v1.0。
