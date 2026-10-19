# 开源 SVS（歌声合成）与「填词成歌」可行性调研

> 调研人：svs-researcher（Team 协作者）｜日期：2026-10-19｜工作目录：F:\Deepseek_harness
> 产出位置：AI音乐处理/05_新方案探讨/02_SVS填词/
> 标注约定：【事实】= 有可验证来源（仓库/发布页/文档/本地实测）；【推断】= 基于经验与间接证据的估计，需实测确认；【风险】= 许可/合规/工程风险。
> 只读原则：本次调研未安装/运行任何源目录软件，未训练模型；所有仓库浅克隆、文档与元数据缓存都放在本目录 _work/ 下。

---

## 0. 结论摘要（先看这段）

**一句话**：2026 年开源世界已经能完成「给定旋律 + 音源（音色）+ 新词 → 唱出来」，但不存在一个“开箱即用”的单一组件；最现实、最可控、许可最干净的主路径仍是 **DiffSinger/OpenUtau 体系（旋律/.ds → 声学模型 → NSF-HiFiGAN 声码器）+ 自建无头渲染与音节分配层**；2026 年新增的 **零样本 SVS（SoulX-Singer）与“改词保旋律”模型（YingMusic-Singer-Plus）** 可以大幅降低门槛、作为快速通道与音色替换器，但分别存在「输入格式偏内部元数据」和「CC-BY/VAE 条款」的约束；全曲生成（ACE-Step 1.5 / YuE2）适合“整首歌重做”，不适合“精确复刻原旋律 + 换词”。

分场景的最短路径：

| 场景 | 推荐路线 | 关键组件 | 主要限制 |
|---|---|---|---|
| 有原曲，只想**换词**并尽量保留原唱旋律/音色 | **YingMusic-Singer-Plus**（2026，CC BY 4.0）或 SoulX-Singer-SVC | 原唱分离 → 旋律 clip + 新词 → 44.1k 人声 | 无音符级控制；VAE 走 Stability 社区许可；长音/复杂转音需人工校对 |
| 有旋律（MIDI/音符），要**指定音色**唱新词 | **SoulX-Singer**（2026-02，Apache-2.0，零样本，42k 小时数据） | 歌词+MIDI/F0 + 10s 音色参考 → 干声 | 输入是内部 JSON 元数据；官方“用户友好 MIDI 输入”仍在 roadmap；需自行做 G2P 与对齐 |
| 要做**产品化/离线/可商用**的填词功能，音色可用通用声库或自训 | **DiffSinger + OpenUtau**（Apache-2.0 / MIT）+ 自建无头渲染 | 旋律提取→G2P→.ds→ONNX 声学+声码器→混音 | OpenUtau 无 CLI（issue #1615）；每个音色需训声库或做 SVC 转换；声码器仓 AGPL 需注意 |
| 只有歌名/歌词/风格，要**整首新歌**（不要求复刻原旋律） | **ACE-Step 1.5**（MIT，cover + 新词，REST API）或 **YuE2**（符号化 ABC 旋律可控，但权重 CC-BY-NC-4.0） | 歌词+风格(+参考音频/ABC) → 整曲 | 旋律不可逐音控制；换词要卡音节/段落；YuE2 权重非商用 |

**回答任务第 6 问（最现实路径）**：
1. **工程落地优先选 DiffSinger 路线**：许可干净（DiffSinger Apache-2.0、OpenUtau MIT、SOME/GAME/TIFA MIT、pypinyin MIT），可完全离线，可逐音符/逐音素编辑，能接任意中文声库；缺口只有三块——**无头渲染器、音节↔音符分配器、最小编辑器**，都是 2~4 人周级别的工作【推断】。
2. **“音色自由”用零样本替代“每音色训 SVS”**：SoulX-Singer（Apache-2.0）零样本 SVS + SoulX-Singer-SVC（Apache-2.0）零样本音色转换，已可覆盖“换音色”需求；RVC 作为社区兜底（MIT，但每音色仍需 10~30 分钟数据训练）。**先 RVC/SVC 再 SVS 不可行**（SVC 输出是音频，不是可用于 SVS 的标注），正确顺序是 **SVS 干声 → SVC/RVC 换音色**。
3. **“已有整首原曲、只想换词”用 YingMusic-Singer-Plus 最快**（一条命令：--melody_audio --target_text），但它不是编辑器级可控，适合“快速出 demo / 批量”。
4. 全曲生成只在“允许旋律变化/只需要风格与歌词”时使用；YuE2 的 ABC 符号规划是唯一“人可读改谱”的整曲方案，但权重 **CC-BY-NC-4.0**，商用需换 ACE-Step 1.5（MIT）。

**成本（粗估，详见 `7）**：零样本推理单句数秒~数十秒（SoulX 2.7GB / YingMusic 2.8GB 权重，建议 ≥8~16GB 显存【推断】）；DiffSinger 自训声库 1~3 小时标注数据 + 单卡 3060 约 1~3 天（多阶段）【推断】；RVC 音色 10~30 分钟数据 + 单卡 3060 约 10~60 分钟【推断/社区经验】；ACE-Step 1.5 整首歌在 3090 上 <10s（README 声称）、显存最低 <4GB（含 offload）。

---

## 1. 调研范围与判据

任务要求的六个维度，本文对应章节：
1. SVS 项目选型 → `3（DiffSinger/OpenUtau/NNSVS/VISinger2/NEUTRINO/UTAU/ENUNU/Sinsy + 2026 新范式）
2. 全曲生成 → `4（ACE-Step、YuE2、DiffRhythm、SongGen）
3. 混合路线（SVS→SVC/RVC）→ `5
4. 旋律/乐谱输入、对齐、拼音/phoneme → `6
5. 工程集成（CLI/ONNX/HTTP/显存/许可）→ `3/`7/`9
6. 编辑器最少交互（对照商业 UX）→ `8

判据（加权）：
- **中文可用性**（普通话/粤语、多音字、儿化、中英混）
- **可控性**（音符级/音素级/参数级；旋律可复刻）
- **许可可商用**（代码与权重分开看）
- **集成成本**（是否有 CLI/ONNX/HTTP、能否无头运行、是否 Windows 友好）
- **数据/训练成本**
- **活跃度与社区**（2026 年是否还在更新）

> 说明：本环境为 Windows；调研以「可被桌面端插件封装」为前提，故 “Linux only / 24GB VRAM” 这类要求会显著扣分。

---

## 2. 2026 年全景对比表

### 2.1 SVS 引擎/声库体系

| 项目 | 类型 | 最新版本/活跃度 | 中文 | 输入 | 接口 | 许可（代码/权重） | 结论 |
|---|---|---|---|---|---|---|---|
| **DiffSinger (OpenVPI)** | 神经 SVS（扩散+声码器） | v2.5.1（2026-01-08）；仓库 2026-10 仍活跃，3.2k★ | ✅ 普通话/粤语；多语言（v2.5 起多词典+音素合并） | 音符+音素+时长+F0；.ds JSON | Python 训练/推理；**ONNX** 导出；OpenUtau 集成 | 代码 Apache-2.0；**声码器仓 openvpi/vocoders 为 AGPL-3.0**【风险】 | **主路径**，生态最完整 |
| **OpenUtau** | 编辑器/渲染宿主（.NET/Avalonia） | 0.1.575 Beta（2026-10-17）、0.1.576.1 Alpha（2026-10-18）；仓库已迁至 openutau/OpenUtau | ✅ 内置 DiffSinger 中文音素器；UTAU 系中文音素器 | .ustx 工程（音符+歌词）；可导出 .ds | **无官方 CLI/无头渲染**（issue #1615）；核心库 OpenUtau.Core 可被 C# 封装 | MIT | 编辑/调参/试听首选；产品集成需补无头层 |
| **DiffScope** | 新一代专业编辑器（C++/Qt/Go） | 开发中（仓库 2025-10 更新） | 依赖 DiffSinger 引擎 | 工程/钢琴卷帘 | 桌面 GUI | 多许可（见仓库 LICENSES 索引） | 可关注，暂不能作为依赖 |
| **NNSVS** | 研究型 SVS 工具包（PyTorch） | 最后提交 2023-10（**停更**），755★ | ⚠️ 社区有中文 recipe，但工具链旧 | 乐谱+音素标签 | Python CLI、可导出 ONNX | MIT | 研究参考，不推荐新项目 |
| **ENUNU** | NNSVS 的 UTAU 插件 | 2025-12 仍有更新，110★ | ⚠️ 取决于训练数据 | UTAU 工程 | Windows 插件 | MIT | 老生态兼容用 |
| **NEUTRINO** | 闭源免费 SVS（日文为主） | 2026-10 仍在更新（Orbit Web 版 2026-10-16） | ❌ 实质日语 | 乐谱类输入+日语歌词 | 闭源 GUI | 闭源免费软件【风险：不可嵌入/不可改】 | 不适用于中文插件 |
| **Sinsy** | DNN/HMM SVS（HTS） | 官方 SourceForge；r9y9 fork 2022 停更 | ❌ 无中文声库 | MusicXML | C++/CLI | 代码开源（HTS 系），**声库许可各自不同**【风险】 | 历史项目，不推荐 |
| **UTAU** | 采样拼接式歌声合成 | 闭源免费（Windows） | ✅ 有大量中文 CVVC/VCV 声库 | .ustx/.ust | 无 API | 闭源免费 | 仅作为 OpenUtau 兼容目标 |
| **VISinger2** | 端到端 SVS 研究代码 | 2023；356★ | ✅（OpenCpop 5h 训练） | 乐谱+音素 | Python | **无 LICENSE 文件（默认保留所有权利）**【风险】 | 仅论文参考 |
| **SoulX-Singer** | **零样本 SVS + SVC**（2026 新） | 代码/权重 2026-02-06；SVC 2026-03-16；HF 2.1k 下载 | ✅ 普通话/英语/粤语（42k 小时） | 歌词+MIDI/F0+**音色参考音频** | Python CLI（python -m cli.inference）、Gradio WebUI、HF Space | **Apache-2.0（代码+权重）** | **零样本主力候选** |
| **YingMusic-Singer-Plus** | **保旋律改词 SVS**（2026 新） | arXiv 2603.24589；HF 模型 2026-04 更新 | ✅ 中/英（可翻译、可 code-switch） | 旋律参考人声 + 新歌词（+可选音色参考） | Python CLI / Gradio / JSONL 批处理 | **CC BY 4.0**（权重），VAE 部分：Stability AI Community License【风险】 | **“改词”最快路径** |
| **Amphion / Vevo2** | 统一语音+歌声生成框架 | Vevo2 2026-03-25；arXiv 2508.16332 | ⚠️ 需自备/微调 | 多种（TTS/SVS/SVC/编辑） | Python | MIT（Amphion） | 研究/二次开发候选 |

### 2.2 全曲生成（Lyrics + Style → Song）

| 项目 | 版本/日期 | 许可（代码/权重） | 旋律可控性 | 与“给定曲子”结合 | 硬件/速度 |
|---|---|---|---|---|---|
| **ACE-Step 1.5** | 仓库 v0.1.8（2026-05-18）；XL 4B（2026-04-02）；技术报告 arXiv 2602.00744 | **MIT / MIT（HF 标注 license:mit）** | 中低：text2music / **cover**（src_audio + 新 lyrics + audio_cover_strength）/ repaint / lego / extract / complete；无逐音符控制 | **可以**：原曲→cover→换词换风格；或 repaint 局部改唱段 | 显存 <4GB（含 offload）~≥24GB 分档（README 表）；3090 上整曲 <10s（声称） |
| **YuE2** | 2026-09-09 发布；权重 m-a-p/YuE2-3B（2026-10-06 更新） | **代码 Apache-2.0 / 权重 CC-BY-NC-4.0**【风险：非商用】 | **高（符号化）**：歌词+风格→ABC 乐谱（可读可改）→渲染；--abc-file + --cot full/melody | **可以且最接近“填词”**：SheetSage2 转谱→改词→cot=melody 重渲染 | **24GB BF16、Linux、48kHz**；单曲分钟级【推断】 |
| **DiffRhythm** | v1.2（2025-05-09）；DiffRhythm 2 论文 2025-10 | Apache-2.0（代码）；HF 卡片未标 license | 低：歌词+风格+参考音频；无旋律/音符条件 | 弱：只能靠参考音频风格迁移 | 显存中等【推断】；285s 全曲 |
| **SongGen** | ICML 2025；训练代码 2025-07 | Apache-2.0 | 低：歌词+描述文本+可选参考音色；无旋律条件 | 弱 | 单卡可跑【推断】 |
| Mureka（商业）等 | 闭源 API | 商业条款 | 不可控/不可本地 | 不适用开源插件 | 云服务 |

> 结论：全曲生成中，只有 **YuE2（ABC）** 与 **ACE-Step cover** 提供“和给定曲子结合”的入口；前者权重 NC，后者 MIT 但可控性弱。二者都不能替代“逐音符合成”。

---

## 3. SVS 路线详解（重点）

### 3.1 DiffSinger（OpenVPI 维护版）——事实与接口

**仓库与版本**【事实】
- https://github.com/OpenVPI/DiffSinger ：Apache-2.0，3222★，默认分支 main，调研时最近推送 2026-10-19。
- 最新 Release：**v2.5.1（2026-01-08，重要 bug fix）**；v2.5.0（2025-04-15，“Universal multilingual support + LYNXNet 解码器 + RoPE”）；v2.4.0（2024-07-13，Rectified Flow + 谐噪分离特征）。
- 文档：仓库 docs/GettingStarted.md、docs/BestPractices.md、docs/ConfigurationSchemas.md；中文导航 https://diffsinger.com 。

**能力**【事实】
- 声学模型：音素序列+音素时长+F0（+可选 variance 参数/语言 id/说话人 id）→ 44.1kHz mel → NSF-HiFiGAN 声码器 → 波形。
- 方差模型（variance）：预测/控制 **时长、音高曲线、能量、气声（breathiness）、voicing、tension**（v2.3 起 voicing/tension），是“自动生成可编辑参数”的关键。
- 多语言/多词典（v2.5+）：dictionaries: {zh, ja, en...} + merged_phoneme_groups + use_lang_id，支持中英日混合与音素合并。
- 44.1kHz（相对原版 24kHz 提升）；支持多说话人、key shift/speed 嵌入、浅扩散加速。

**数据与训练流程**【事实】（docs/BestPractices.md、docs/GettingStarted.md）
1. 原始数据：raw/wavs/*.wav + raw/transcriptions.csv，必填列 name, ph_seq, ph_dur（声学）；方差模型另需 note_seq, note_dur, note_slur（可选 f0 等）。
2. 预处理：python scripts/binarize.py --config my_config.yaml
3. 训练：python scripts/train.py --config my_config.yaml --exp_name my_experiment --reset（断点续训；DDP 多卡）
4. 推理：python scripts/infer.py acoustic my_song.ds --exp ...（方差模型用 variance）
5. 导出：python scripts/export.py acoustic|variance|nsf-hifigan → **ONNX**
6. 词典：dictionaries/opencpop-extension.txt（普通话，**拼音→音素，无声调**；如 ai → ai、ba → b a），全局音素 SP/AP，自定义 extra_phonemes（如 EP）。
7. 数据规模建议：**官方文档未给硬性小时数**；社区实践 1~3 小时干净、标注准确的演唱即可训出可用声库【推断；对照：VISinger2 用 OpenCpop≈5h 训练 500k steps，DiffSinger 社区声库普遍 1~5h 数据】。

**.ds 格式**【事实，本地实测】
- JSON 数组，每个元素是一段（可多段）：
  - offset（秒）、text（空格分词的字/词）、ph_seq（音素）、ph_dur（秒）、ph_num（每个词含几个音素）
  - note_seq（音名如 C4/rest）、note_dur、note_slur（0/1，连音）
  - f0_seq 及可选参数曲线（f0_timestep 等，OpenUtau 导出时带）
- 仓库 samples/ 自带 7 个中文示例：00_我多想说再见啊.ds … 07_春江花月夜.ds（如 03_撒娇八连.ds 含 AP/SP、滑音 note_slur）。
- **本地验证**：我写了最小生成器 _work/proto_midi2ds.py（仅标准库），输入“音符+拼音音节”JSON、用 opencpop-extension.txt 查音素，生成 .ds 并自检通过：notes=8 phonemes=15 total_dur=4.750s（输出 _work/out_demo.ds）。说明“旋律+歌词→.ds”这一层几乎零依赖、可快速自研。

**ONNX 接口（供无头渲染器）**【事实，读 deployment/exporters/*.py】
- 声学模型输入：tokens（音素 id，int64）、durations、f0；若启用则追加 energy/breathiness/voicing/tension（variance_embed_list）、gender、velocity、spk_embed、languages；输出 mel。
- 声码器输入 mel → 输出波形（44.1kHz，mel_base=e, mel_scale=slaney；默认 pc_nsf_hifigan_44.1k_hop512_128bin_2025.02）。
- dsconfig.yaml 字段（OpenUtau 侧解析，见 OpenUtau.Core/DiffSinger/DiffSingerConfig.cs）：phonemes/acoustic/vocoder/speakers/hiddenSize/useKeyShiftEmbed/useSpeedEmbed/useEnergyEmbed/useBreathinessEmbed/useVoicingEmbed/useTensionEmbed/use_lang_id/dur/dur_type/linguistic/pitch/variance/predict_dur/predict_energy/predict_breathiness/predict_voicing/predict_tension/use_expr/use_note_rest/sample_rate 等。
- 性能线索：旧版 ONNX 导出手册称 **ONNXRuntime 比 PyTorch 快 25%~50%**（https://github.com/openvpi/DiffSinger/blob/eecb002f7ed61e5da1f49236a3dce3e0d55b4954/docs/README-SVS-onnx.md ，PyTorch 1.8 时代；新版未再标数字）【事实（旧文档）+推断（新版本同量级）】。

**声库目录结构（OpenUtau 版）**【事实，读 C# 源码】
- dsconfig.yaml、phonemes.txt、dsdict.yaml（分语言 dsdict-zh.yaml 等）、dspitch/dsconfig.yaml、dsvariance/dsconfig.yaml、声学/方差/vocoder ONNX 文件。
- OpenUtau 内置 DiffSingerChinesePhonemizer 等 20+ 音素器（OpenUtau.Core/DiffSinger/Phonemizers/），说明“歌词→音素”在宿主层已解决。

**许可**【事实】DiffSinger 代码 Apache-2.0；但 **OpenVPI 官方声码器仓 openvpi/vocoders 为 AGPL-3.0**（2025-02 更新）。若插件以网络服务方式分发/修改声码器代码，AGPL 义务会传导【风险；建议：仅“分发未修改的 ONNX 权重”并核对权重许可，或换 MIT 的 openvpi/SingingVocoders 自训声码器】。社区声库许可各自独立（见 `9）。

### 3.2 OpenUtau——编辑器宿主与集成缺口

【事实】
- 仓库：https://github.com/openutau/OpenUtau （原 stakira/OpenUtau 已重定向；README 仍写 stakira 链接），MIT。
- 版本节奏：**0.1.575 Beta（2026-10-17）**、**0.1.576.1 Alpha（2026-10-18）**（release atom 实测）。
- 0.1.575 新特性：警告中心、**DiffSinger group-aware duration 模型**（按 dsconfig.yaml 的 dur_type 选择 word_div/word_dur 输入）、UI/主题修复、WORLDLINE-R1.1 渲染器等。
- 支持 UTAU 声库、DiffSinger ONNX 声库、ENUNU、Vogen；OpenUtau.Core 为可复用 .NET 库；OpenUtau.Plugin.Builtin 内置大量音素器（中文 CVVC/CVVPlus/VCV 等）。
- **无 CLI/无头模式**：当前解决方案文件仅含 OpenUtau.Core、Plugin.Builtin、Test、UiTest、主程序；Program.cs 的 Main(args) 不解析命令行；社区 issue **#1615 “Export WAV from USTX file without launching UI”**（请求无头导出）【事实】。
- 导出能力：BestPractices 提到 OpenUtau 可导出 .ds 文件供 DiffSinger 推理【事实】。

**结论**：OpenUtau 适合“人工编辑/试听/调参”和“导出 .ds”；产品化自动渲染必须自建：
- 方案 A（推荐）：Python + onnxruntime 直接跑声学+声码器（输入契约见 `3.1），无 GUI 依赖、跨平台、可做 HTTP 服务。
- 方案 B：用 C# 包一层 OpenUtau.Core（MIT）做无头 CLI（工程量中等，需要 .NET 8 + Avalonia 依赖剥离）。
- 方案 C：改造 DiffSingerMiniEngine（AGPL-3.0，2024 停更；python server.py 提供最小 ONNX 推理服务，支持 MIDI-less 模式）——许可与维护性差【风险】。

### 3.3 DiffScope（OpenVPI 新一代编辑器，WIP）

【事实】https://github.com/diffscope/diffscope-project ：C++（前端核心）+ Qt QML（GUI）+ Golang（后端），README 明确 “under development”；已完成：应用壳、编排/钢琴卷帘、基础音频处理、**音素与参数编辑器**、音频播放、**合成引擎集成**。是“DiffSinger 的现代专业编辑器”方向，值得跟踪，但不能作为当前依赖。

### 3.4 其他 SVS 体系的结论（简短）

- **NNSVS**（MIT，r9y9/nnsvs，2023-10 后停更）：研究型，配方偏日语/英语；中文需自建 recipe + 自行维护旧 PyTorch 依赖。**不推荐新产品**，但它的语料准备脚本（nnsvs-db-converter）在社区仍在用。
- **ENUNU**（MIT，2025-12 更新）：NNSVS 的 UTAU 插件，用于“UTAU 生态兼容”，无产品价值。
- **NEUTRINO**：闭源免费，实质日语，2026 年仍在更新（Orbit Web 版 2026-10-16，studio-neutrino.com）。不可嵌入、不可改，**排除**。
- **Sinsy**：HTS DNN/HMM，MusicXML 输入，官方以日语为主，无中文声库；C++ 老代码（r9y9 fork 2022 停更）。**排除**。
- **UTAU**：闭源免费采样拼接引擎；只能通过 OpenUtau 兼容。**排除**（除了声库资产兼容）。
- **VISinger2**：官方实现 https://github.com/zhangyongmao/VISinger2 ，**无 LICENSE 文件**（默认保留所有权利），2023 停更；README 记录 **OpenCpop 上训练 500k steps、batch 16**。**仅研究参考**【风险】。

### 3.5 2026 新范式一：SoulX-Singer（零样本 SVS + SVC）

【事实，来源：README/arXiv/HF】
- 仓库 https://github.com/Soul-AILab/SoulX-Singer ；论文 arXiv:2602.07803（v1 2026-02-08，v2 2026-03-16）；HF Soul-AILab/SoulX-Singer（model.pt 2.69GB、model-svc.pt 2.67GB，license: apache-2.0）；在线 Demo/MIDI Editor（HF Spaces）。
- 能力：**零样本**（未见过的音色，用参考音频克隆音色）+ **歌词+MIDI（score 模式）或 F0（melody 模式）** 控制音高/节奏；支持**改词保韵律**（singing voice editing）；训练数据 **42,000+ 小时**（普通话/英语/粤语）。
- SVC 子模型（2026-03-16）：**无需歌词/MIDI 转写**的 audio→audio 音色转换，保旋律/节奏/歌词。
- 推理入口：python -m cli.inference --prompt_wav_path ... --prompt_metadata_path ... --target_metadata_path ... --control score --fp16；WebUI python webui.py。
- 元数据 JSON 结构（example/audio/music.json 实测字段）：index, language, time[], duration[], text[], phoneme[], note_pitch[], note_type[], f0[]；音素带声调（如 zh_zhi3）；note_pitch 为 MIDI 整数，休止为 0。
- 预处理管线（preprocess/README.md）：人声分离+去混响（Music-Source-Separation-Training / mel-band-roformer-karaoke / dereverb）→ F0（RMVPE）+VAD → 歌词 ASR（Paraformer / Parakeet）→ 音符转写（ROSVOT）→ 元数据；提供 **MIDI Editor**（HF Space）与 meta2midi / midi2meta 工具，官方明确“自动对齐不完美，建议人工校正”。
- 依赖：Python 3.10、torch 2.2.0、funasr、g2pM、ToJyutping、mido、pretty_midi、parselmouth、pyworld 等（requirements.txt）。
- 许可：**Apache-2.0（代码与权重）**，附使用免责声明（不得未经授权模仿真人）。

**评价**：这是目前“零样本 + 音符级控制 + 中文”最完整开源方案；短板是**没有面向用户的 MIDI 输入格式**（roadmap 未完成）、元数据需自己生成/清洗、显存未在 README 标注（2.7GB fp16 权重，估计 8~16GB【推断】）。

### 3.6 2026 新范式二：YingMusic-Singer-Plus（保旋律改词，产品化最快）

【事实】
- 仓库 https://github.com/ASLP-lab/YingMusic-Singer-Plus ；论文 arXiv:2603.24589；HF ASLP-lab/YingMusic-Singer-Plus（model.safetensors 2.78GB；完整 12.16GB 含分离/VAE checkpoint）。
- 任务定义即“填词”：**可选音色参考 + 提供旋律的演唱片段 + 修改后的歌词 → 44.1kHz 歌声，保留原旋律**；无需歌词-MIDI 对齐（annotation-free）。
- 支持 6 类改词：局部/整体替换、插入、删除、中英翻译、code-switch；发布 **LyricEditBench** 基准（旋律保持的改词评测，基于 GTSinger）。
- CLI 极简：
  python infer.py --ref_audio 音色参考.wav --melody_audio 旋律参考.wav --ref_text “原词|分句” --target_text “新词|分句” --output out.wav
  可选 --separate_vocals --mix_accompaniment（内置人声分离与伴奏混合）；另有 batch_infer.py（JSONL）与 app_local.py（Gradio）。
- 要求：输入必须是**纯人声**（可用内置分离）；Python 3.10；GPU 推荐；词按 | 分句（对旋律分句）。
- 许可：**代码与权重 CC BY 4.0**；**例外**：Stable Audio 2 VAE 及其推理代码来自 Stable Audio Open，走 **Stability AI Community License**【风险：商用需核对年收入门槛等条款】。

**评价**：“已有歌曲/人声，换词再唱”这条需求，它是最短路径（一条命令），且有专门 benchmark；但不是逐音符编辑器，长音/转音/多音节分配错位时需要人工或换方案。

---

## 4. 全曲生成路线（补充/替代）

### 4.1 ACE-Step 1.5（MIT，工程最友好）
【事实】
- 仓库 https://github.com/ace-step/ACE-Step-1.5 ；论文 arXiv 2602.00744（2026-02）；HF ACE-Step/Ace-Step1.5（license: mit，77k+ 下载）；GitHub 13.8k★。
- 发布：v0.1.5（2026-03-25）→v0.1.6（2026-04-03，XL 4B DiT 发布 2026-04-02）→v0.1.7（2026-04-24）→**v0.1.8（2026-05-18）**。
- 任务类型：text2music / cover / repaint / lego / extract / complete；**cover 支持 src_audio + 新 lyrics**（文档原文：lyrics: New lyrics (if changing vocals)）+ audio_cover_strength（1.0 强贴原结构，0.2 风格迁移）。
- 接口：Gradio（7860）、**REST API（8001，POST /release_task → POST /query_result）**、Python API、CLI；支持 BPM/Key/时长(10~600s)/语言等参数；LRC 时间戳生成。
- 性能：README 称 3090 上整曲 <10s、A100 <2s；显存分档：≤6GB 可跑（2B turbo + int8 + CPU offload），8~16GB 主流，≥24GB 跑 4B LM/XL。
- 局限：**旋律不可逐音控制**（cover 只能“保持结构”），换词需歌词音节/段落与原曲匹配；生成结果的“换词对齐”质量随机，需要多条候选挑选【事实+推断】。

### 4.2 YuE2（符号规划，最像“改谱填词”，但权重非商用）
【事实】
- 仓库 https://github.com/multimodal-art-projection/YuE ；YuE2 发布 2026-09-09（12.3k★）；论文 arXiv 2609.33757；HF m-a-p/YuE2-3B 权重 **CC-BY-NC-4.0**（代码 Apache-2.0）；转谱模型 m-a-p/SheetSage2 亦 **CC-BY-NC-4.0**。
- 工作方式：歌词+风格 → **ABC 乐谱**（旋律+和弦，可读可改）→ 渲染 48kHz 立体声整曲；--cot full/melody；--abc-file 可直接给谱。
- cover 流程：SheetSage2 转谱（--melody-only）→ 人工核对 ABC → 新歌词+目标风格 → cot=“melody” 重新渲染（无需为 cover 微调）。
- 硬件：**Linux、Python 3.12、NVIDIA BF16、24GB VRAM**；单曲分钟级【推断】。
- 评价：唯一“白盒/可编辑乐谱”的整曲方案，适合“整首重编曲+换词”；但 **NC 权重**不适合商业产品，且不是逐音素/逐音符可控的 SVS。

### 4.3 DiffRhythm / SongGen
【事实】DiffRhythm：Apache-2.0（2025-03-07 起），v1.2（2025-05-09），285s 全曲，歌词+风格/参考音频，无旋律条件；DiffRhythm 2 论文（arXiv 2510.22950，2025-10）但未见权重发布【推断】。SongGen：ICML 2025，Apache-2.0，歌词+描述+可选参考音色，无旋律条件，训练/推理代码已释出。二者都只能“按词生成新旋律”，不适合复刻给定旋律。

---
