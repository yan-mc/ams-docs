## 5. 混合路线：SVS 干声 → 音色转换（SVC/RVC）

### 5.1 正确顺序与可行性
- **可行且是社区主流**：DiffSinger/OpenUtau 先用**通用中文声库**唱出干声（音素级正确、节奏精确），再用 **SVC/RVC** 换成目标音色。这样**不必为每个音色训练 SVS 声库**，只需为目标音色准备 10~30 分钟干净人声训练 SVC（或直接用零样本 SVC）。
- **“先 RVC 后 SVS”不可行**：SVC/RVC 输出是音频，不产生音符/音素标签；且音色转换后再做 SVS 属于“用转换后的音频再训声学模型”，误差与成本都更大【推断，有明确逻辑依据】。

### 5.2 工具与许可（2026-10）
| 工具 | 类型 | 许可 | 数据/训练 | 状态 |
|---|---|---|---|---|
| **SoulX-Singer-SVC** | 零样本 SVC（audio→audio，保旋律/节奏/歌词） | **Apache-2.0** | 无需训练，参考音频即可；**无需歌词/MIDI 转写** | 2026-03-16 发布，最新 |
| **RVC**（Retrieval-based-VC-WebUI） | 训练式 SVC | MIT，38.8k★，2026-08 仍更新 | 每音色 10~30 分钟干净人声，3060 约 10~60 分钟【推断/社区经验】；8GB 显存可训 | 生态最成熟，整合包多 |
| **Seed-VC** | 零样本 VC | GPL-3.0，**仓库已归档（2025-04）** | 无需训练 | 不推荐新项目（GPL + 停更） |
| **so-vits-svc** | 训练式 SVC | **AGPL-3.0**，仓库进入归档状态 | 每音色训练 | 遗留项目 |
| **Vevo2 / Vevo1.5（Amphion）** | 统一框架（SVS/SVC/编辑/风格迁移） | MIT（Amphion） | 零样本/可控 | 2026-03-25 / 2025-04-12，研究向 |

### 5.3 音质损失与失败模式【推断 + 部分事实】
- 已知风险：气声/呼吸噪声、齿音（sibilance）、高频细节、共振峰（formant）偏移、长音稳定性、颤音被“抹平”、音素边界模糊（辅音被吞）。RVC 类模型对“训练集中未出现的唱法/极端音高”更易出电音/沙哑。
- 证据线索：RVC 使用 hubert/whisper 内容特征 + F0，内容编码会损失部分发音细节；so-vits-svc README 亦提示响度匹配不当会伤音质（本调研读过其 README）。零样本 SVC（SoulX/Vevo）通常比“小数据 RVC”更稳，但音色相似度可能略低【推断，需 A/B】。
- 缓解：SVS 干声先做轻压缩/EQ 再转音色；一次转换不超过 ±5 半音；保留原 F0；转换后用 F0 RMSE + ASR 回读（CER）做客观筛选；最终以 A/B MOS 小样本主观评测。

### 5.4 成本对比
- 每新增一个音色：训 RVC ≈ 10~60 分钟（3060）+ 10~30 分钟数据；用零样本 SVC ≈ 数秒/句（无需训练）；训 DiffSinger 声库 ≈ 1~3 天（3060）+ 1~3 小时标注数据【均为推断】。
- 因此产品策略：**少量主推音色训 DiffSinger；任意用户音色走零样本 SVS/SVC**。

---

## 6. 旋律/乐谱输入、对齐、拼音与音素

### 6.1 输入方式（从易到难）
1. **已有 MIDI/ustx**：直接解析音符（OpenUtau .ustx 是 YAML/JSON 风格；MIDI 用 mido/pretty_midi）。
2. **已有歌曲音频**：人声分离 → F0/MIDI 提取 → 音符切分 → 人工校正（编辑器）。
3. **手绘 piano-roll**：最小实现是“网页/桌面的音符栅格”。
4. **哼唱/清唱**：F0 提取（RMVPE/pYIN）+ 音符切分。

### 6.2 工具链（均为可集成的开源组件）
| 环节 | 工具 | 许可 | 关键事实 |
|---|---|---|---|
| 人声分离 | UVR5 / MDX23C / BS-Roformer（Music-Source-Separation-Training, ZFTurbo） | 多为 MIT/学术许可，模型各自条款 | SoulX/YingMusic 官方预处理均使用该类模型（README 致谢）【事实】 |
| F0 提取 | **RMVPE**（yxlllc/RMVPE） | 仓库未标 LICENSE（谨慎）【风险】 | DiffSinger/SoulX 官方推荐；也可用 librosa pYIN（ISC）或 torchcrepe（MIT）【事实/推断】 |
| 歌唱 MIDI 转写 | **SOME**（MIT，2024）、**GAME**（MIT，2026-08 更新，SOME 后继） | MIT | SOME：i5-12400 上 9× 实时、3080Ti 上 300×；3 小时数据可训练；输出非整数音高适合方差标注（README）；GAME：D3PM 生成式边界提取，抗噪/抗混响/带伴奏，输出 note_seq/note_dur/note_slur 并与 DiffSinger 数据集格式互通 |
| 复音转写（备选） | basic-pitch（Spotify，Apache-2.0） | Apache-2.0 | 通用 MIDI 转录，人声也能用，复音场景更稳【推断】 |
| 歌词强制对齐 | **TIFA**（OpenVPI，MIT，2026-10 活跃；v1.0.0 模型） | MIT | 专为“语音/歌声 + 带伴奏/噪声”设计，音素级 TextGrid，内含 g2pflow 多语言 G2P；支持 pronunciation scoring |
| 对齐（传统） | MFA（MIT，2026-10 活跃） | MIT | 需要发音词典与声学模型；歌声需专门模型；批量稳定 |
| 对齐（快速） | WhisperX（BSD-2） | BSD-2 | 词级时间戳，非音素级；歌声准确率一般【推断】 |
| 中文 G2P | **pypinyin**（MIT，2026-10 活跃）、g2pM（SoulX 使用）、**g2pflow**（OpenVPI，MIT，2026-09） | MIT | **本地实测** pypinyin：重庆→chong2 qing4、银行行长→yin2 hang2 hang2 zhang3、音乐→yin1 yue4、快乐→kuai4 le4、长大了→zhang3 da4 le（多音字上下文正确；测试脚本见附录） |
| 中文分词/断句 | jieba（MIT）等 | MIT | 分词后按字/词分配音符；儿化/轻声需自定义规则【推断】 |

### 6.3 本地小实验（可复现）
1. **.ds 生成器原型**：_work/proto_midi2ds.py + _work/demo_melody.json → _work/out_demo.ds（8 音符/15 音素/4.75s，自检 len(ph_seq)=len(ph_dur)=sum(ph_num)、时长和一致）。
2. **pypinyin 多音字**：_work/pylibs（pip install --target，纯 Python 轻量包）+ 命令行实测（结果见上表）。
3. **OpenUtau/.ds 结构**：读仓库 samples/*.ds、OpenUtau.Core/DiffSinger/*.cs 交叉验证（见 `3.1/`3.2）。
4. **未做**（受“不安装重型依赖/不训练”限制）：DiffSinger ONNX 实际推理、SoulX/YingMusic 实际出音、音质 A/B —— 已在 `10 列为必做验证实验。

---

## 7. 推荐管线与缺口清单

### 7.1 三条候选管线

**管线 A（推荐主路径，工程化/可商用/可编辑）**
@@@
旋律来源（MIDI / 分离人声 / 手绘）
  → [GAME/SOME] 音符切分 + F0（或直接 MIDI）
  → 人工校正（最小 piano-roll，见 `8）
  → 歌词文本 → pypinyin/jieba 分词注音 → 音节-音符分配（自动 + 手工拖动）
  → 生成 .ds（音符/音素/时长/滑音；可选 variance 参数）
  → DiffSinger 声学 ONNX（+ variance ONNX 自动生成 音高/能量/气声曲线）
  → NSF-HiFiGAN 声码器 ONNX → 干声
  → （可选）SoulX-Singer-SVC 换目标音色
  → 与伴奏（UVR 分离）对齐/混音 → 导出 wav + 工程
@@@
优点：许可最干净（Apache/MIT）、完全离线、逐音符可控、可接任意中文声库；缺点：需自研无头渲染器、音节分配器、最小编辑器（≈2~4 人周【推断】）。

**管线 B（零样本，最快接入，少训练）**
@@@
歌词 + MIDI/音符 + 10s 目标音色参考
  → SoulX-Singer（score 模式，生成“演唱元数据 JSON”）
  → 干声 → 混音
@@@
适合“用户上传音色样本即用”；音色相似度与稳定性需 A/B 验证；许可 Apache-2.0 最友好。

**管线 C（已有原曲，直接改词）**
@@@
原曲 → 人声分离（内置）→ 旋律参考 clip + 原词 + 新词
  → YingMusic-Singer-Plus → 新干声（保旋律）→ 混伴奏
@@@
一条命令出结果，适合 demo/批量；不是编辑器。

> **建议的产品形态**：以 A 为主（填词编辑器 + 通用声库），把 B/C 作为“加速/兜底模式”一键切换；当用户没有 MIDI 时用 C 先出草稿，再进 A 精修（共享同一套歌词/音节数据结构）。

### 7.2 缺口清单（按优先级）

| P | 缺口 | 说明/建议 | 工作量【推断】 |
|---|---|---|---|
| P0 | **无头 DiffSinger 渲染器** | Python + onnxruntime 实现 声学+variance+vocoder 三级推理，读 dsconfig.yaml/dsdict；输出 wav；提供 HTTP API | 1~2 周 |
| P0 | **音节↔音符分配器** | 中文按字/词切分，处理一字多音（如“儿化”）、一音多字（连音/滑音 note_slur）、休止 SP/AP；自动按数量 + 手动拖动 | 1 周 |
| P0 | **中文声库包管理 + 许可元数据** | 声库下载/校验/缓存；记录每个声库的许可证与署名要求；避免默认捆绑 NC 声库 | 3~5 天 |
| P1 | **最小编辑器** | piano-roll + 歌词行 + 音素/参数面板 + 试听（`8） | 2~3 周 |
| P1 | **旋律提取与校正** | 集成 GAME（GPU 可选），失败回退 basic-pitch/pYIN；提供“音符吸附/量化” | 3~5 天 |
| P1 | **音色替换** | 优先 SoulX-Singer-SVC（Apache）；RVC 作为可选插件 | 3~5 天 |
| P1 | **混音/对齐** | 伴奏分离（UVR/BS-Roformer）+ BPM 对齐 + 响度匹配 | 3~5 天 |
| P2 | **质量评估** | F0 RMSE、ASR CER、MOS 小样本、批量 A/B | 3 天 |
| P2 | **全曲生成兜底** | ACE-Step 1.5 REST（MIT）；YuE2 仅内部研究用（NC） | 2~3 天 |
| P2 | **工程互通** | OpenSVIP（openvpi.github.io，支持 SVP/X Studio/ACE Studio/MIDI/LRC 等工程格式转换） | 视需求 |

### 7.3 训练/推理成本表

| 项 | 数据 | 硬件 | 时间（估） | 说明 |
|---|---|---|---|---|
| DiffSinger 声库（声学+方差） | 1~3h 标注演唱 | 单卡 3060 12GB | **1~3 天**【推断】 | 多阶段：binarize/训练/导出；可用社区一键脚本（如 ds-train-scripts） |
| DiffSinger 推理 | - | CPU/GPU | GPU 实时率约 0.05~0.3；CPU 约 0.5~2【推断，须实测】 | ONNX 比 PyTorch 快 25~50%（旧官方文档） |
| 声码器微调 | 同声库 | 3060 | 数小时【推断】 | 官方强烈建议为每个声库微调 |
| SoulX-Singer 零样本推理 | 参考音频 10s | GPU（8~16GB【推断】） | 每句数秒【推断】 | 无需训练 |
| YingMusic-Singer-Plus | 纯人声旋律 clip + 新词 | GPU（8~16GB【推断】） | 每句数秒~十几秒【推断】 | 内置分离/混伴奏 |
| RVC 音色 | 10~30min 干净人声 | 3060 8GB+ | 10~60 分钟【推断】 | 每个音色一个模型 |
| ACE-Step 1.5 整曲 | 歌词+风格(+src_audio) | ≤6GB 可行 / 主流 8~24GB | 3090 <10s（声称） | REST/CLI |
| YuE2 整曲 | 歌词+ABC/风格 | 24GB BF16、Linux | 分钟级【推断】 | 权重 NC |
| GAME/SOME 转谱 | 人声 | CPU 可（SOME 9× 实时） | 每首 <1 分钟 | GPU 更快 |
| TIFA 对齐 | 人声+文本 | GPU 更佳【推断】 | 每首 <1 分钟 | 支持带伴奏 |

---

## 8. 编辑器最少交互（对照 Vocaloid / SynthV / ACE Studio）

### 8.1 商业 UX 参照【事实（公开产品资料）】
- **Synthesizer V Studio 2**：2025-03-21 发布，稳定版 2.2.1（2026-03-02，Wikipedia）；卖点是 AI 声库 + 自然度 + **AI Retakes（一键重唱）** + 丰富的音高/表情参数；支持中文（普通话/粤语）。
- **VOCALOID**：VOCALOID 6（2022）延续“音符+参数”工作流；Wikipedia 已列出 VOCALOID 7 章节（细节以官网为准）。
- **ACE Studio**（acestudio.ai，商业）：AI 歌手 + 歌词编辑 + 参数自动化，是最接近“填词即出歌”的产品形态。
- **DiffScope（开源 WIP）**：参数化编辑器 + 音素编辑 + 引擎集成，说明开源侧共识是“钢琴卷帘 + 音素/参数两栏”。

### 8.2 填词功能的“最少交互”（MVP，建议 10 项）
1. **导入/生成旋律**：MIDI/ustx 导入、或“上传歌曲→自动转谱”（GAME）→ 音符带歌词占位。
2. **歌词输入与分句**：整段中文粘贴 → 自动分词/注音 → 自动按音节数分配到音符；支持 | 分句对齐。
3. **音符级编辑**：拖动音高/时长、增删音符、连音（slur）、休止/呼吸（SP/AP）。
4. **音节分配修正**：每字对应音符数调整（一字多音/多字一音）、拖拽换位。
5. **试听与局部重渲染**：整段/单句渲染（缓存、增量）。
6. **音素面板（可选但有）**：声母/韵母时长、替换音素、查看实际音素序列。
7. **表情控制（3 个滑杆即可）**：气声 breathiness、张力 tension、力度/能量 energy；进阶给“颤音 深度/频率/延迟”。
8. **音高曲线**：自动曲线 + 可拖控制点（滑音/转音）；“重绘（retake）”按钮。
9. **音色选择**：声库下拉 + 零样本参考音频上传（SoulX/YingMusic 模式）。
10. **导出**：干声/混音（含伴奏）/工程文件；署名与许可提示。

> “最少交互”判断标准：普通用户从“粘贴歌词”到“听到第一句”≤3 次点击；专业用户永远可以下钻到音素/参数层。这与 SynthV/ACE Studio 的“简单模式 + 专业模式”分层一致。

---

## 9. 许可证风险清单【风险】

| 组件 | 许可 | 风险/要求 |
|---|---|---|
| DiffSinger 代码 | Apache-2.0 | 可商用，需保留版权/NOTICE |
| **openvpi/vocoders（声码器）** | **AGPL-3.0** | 若修改/网络分发，可能要求开源整个服务；建议“仅分发未修改 ONNX”并核对权重许可，或改用 MIT 的 openvpi/SingingVocoders |
| OpenUtau | MIT | 可商用 |
| DiffSingerMiniEngine | AGPL-3.0 | 同上；且 2024 停更 |
| SOME / GAME / TIFA / g2pflow | MIT；MakeDiffSinger 为 BSD-3 | 可商用（保留声明） |
| **社区中文声库（示例）** | **各自许可** | 例：Tiger DiffSinger **CC BY-NC-ND 4.0 + Commons Clause**（禁商用/禁演绎，商用需购买许可）；绮萱 Qixuan 声库仓库含**中文优先的 Terms of Use**（需逐条遵守）；多数社区声库“非商用/需授权/禁止用于特定内容” |
| **SoulX-Singer** | Apache-2.0（代码+权重） | 可商用；但有“不得未经授权模仿真人”的使用声明 |
| **YingMusic-Singer-Plus** | CC BY 4.0（代码+权重）；**VAE Stability AI Community License** | CC-BY 需署名；Stability 条款有商用收入门槛等要求；上线前必须法务确认 |
| **YuE2 权重 / SheetSage2** | **CC-BY-NC-4.0** | **禁止商业使用**；仅内部研究/评测 |
| ACE-Step 1.5 | MIT（代码+权重） | 可商用 |
| DiffRhythm / SongGen | Apache-2.0 | 可商用（注意 HF 卡片未标权重许可，建议核对） |
| RVC | MIT | 可商用（训练数据版权自负） |
| SoulX-Singer-SVC | Apache-2.0 | 可商用 |
| Seed-VC | GPL-3.0 | 传染性；已归档 |
| so-vits-svc | AGPL-3.0 | 传染性；已归档 |
| NNSVS / ENUNU / pypinyin / basic-pitch / WhisperX / MFA | MIT/MIT/MIT/Apache/BSD-2/MIT | 可商用 |
| RMVPE | **无 LICENSE 文件**（谨慎） | 建议换成有明确许可的 F0 模型或自训 |
| **声音权/曲版权** | - | 输出歌声的商用还涉及“音源音色授权”与“词曲版权”；产品需声明与过滤（如禁止模仿真人） |

---

## 10. 验证实验建议（按优先级）

**P0（1~2 天内可完成，直接决定路线）**
1. **DiffSinger 端到端**：下载 1 个中文 DiffSinger 声库（ONNX）→ 用我们的 .ds 原型跑 ONNX 声学+声码器 → 得到干声；测单句 RTF（CPU/GPU）。
2. **SoulX-Singer 零样本**：用官方示例元数据 + 中文参考音频跑 cli.inference；把一段 MIDI+新词转成其 metadata JSON，检查音准与吐字。
3. **YingMusic-Singer-Plus 改词**：取一段中文人声（先用内置分离），原词→新词，检查旋律保持度、长音、多音节。
4. **OpenUtau 人工路径**：编一段 .ustx 用 DiffSinger 声库渲染，导出 .ds，与 P0-1 结果对比（验证我们自研 .ds 生成器语义一致）。

**P1（1 周）**
5. 用 GAME 从真实歌曲人声提取 MIDI，评估“自动转谱 + 人工校正”所需时间。
6. 用 TIFA 对“人声 + 原词”做音素对齐，评估在带伴奏/混响下的可用性。
7. 客观评测：F0 RMSE、ASR 回读 CER；主观 5 人×10 句 A/B（DiffSinger vs SoulX vs YingMusic）。
8. 混音流水线：UVR/BS-Roformer 分离 → 对齐 → 响度匹配（pyloudnorm）。

**P2（2~4 周）**
9. 最小编辑器原型（piano-roll + 歌词 + 参数 3 滑杆 + 试听）。
10. 声库许可元数据与“商用/非商用”开关（避免默认捆绑 NC 声库）。
11. ACE-Step cover 兜底链路（REST）与质量抽检。

---

## 11. 结论（对应任务第 6 问）

**“给定一首曲子的旋律 + 音源 + 新词 → 唱出来”，2026 年开源世界的最现实路径**：

1. **要可控、可商用、可产品化 → DiffSinger/OpenUtau 体系**（Apache-2.0/MIT），补 3 个组件即可闭环：
   - 无头渲染器（Python + onnxruntime，按 `3.1 的 ONNX 契约）；
   - 音节-音符分配器（pypinyin/jieba + 手工校正）；
   - 最小编辑器（piano-roll + 歌词 + 3 个参数滑杆）。
   训练成本：通用中文声库可现成获取（注意许可），自训声库 1~3 小时数据 + 单卡 1~3 天【推断】；推理可在消费级 GPU 甚至 CPU 上跑。
2. **要“零训练、任意音色” → SoulX-Singer（Apache-2.0）零样本 SVS + SoulX-Singer-SVC 换音色**；它把“为每个音色训 SVS”变成“传 10 秒参考音频”。
3. **要“已有歌曲只换词” → YingMusic-Singer-Plus（CC BY 4.0）**，一条命令、保旋律、带伴奏混合；适合草稿/批量，精修回到路线 1。
4. **要整首新歌（旋律可变） → ACE-Step 1.5（MIT，REST，快）**；YuE2 的 ABC 符号规划最接近“人改谱再唱”，但 **权重 CC-BY-NC-4.0 不可商用**。
5. **混音色路线**：SVS 干声 → SVC/RVC（顺序不可颠倒）；零样本 SVC 优先于每音色训 RVC。
6. **主要风险**：声码器/部分工具 AGPL-3.0、社区声库普遍 NC、YuE2/SheetSage2 权重 NC、YingMusic 的 VAE 走 Stability 条款、音色与词曲版权；以及 **OpenUtau 无 CLI** 这一工程缺口。

---

## 附录 A：版本/日期速查（均为调研时实测）

| 项目 | 版本/日期 | 来源 |
|---|---|---|
| DiffSinger | v2.5.1 2026-01-08；v2.5.0 2025-04-15；v2.4.0 2024-07-13 | GitHub releases.atom（openvpi/DiffSinger） |
| OpenUtau | 0.1.576.1 Alpha 2026-10-18；0.1.575 Beta 2026-10-17 | releases.atom（openutau/OpenUtau） |
| ACE-Step 1.5 | v0.1.8 2026-05-18；v0.1.5 2026-03-25；XL 4B 2026-04-02 | releases.atom（ace-step/ACE-Step-1.5）+ README |
| YuE2 | 发布 2026-09-09；skill 1.2.0 2026-09-25 | releases.atom（multimodal-art-projection/YuE） |
| DiffRhythm | v1.2 2025-05-09 | Readme.md（ASLP-lab/DiffRhythm） |
| SoulX-Singer | 代码/权重 2026-02-06；SVC 2026-03-16；arXiv v2 2026-03-16 | README + HF + arXiv |
| YingMusic-Singer-Plus | arXiv 2603.24589；HF 2026-04-09 更新 | HF API |
| Synthesizer V Studio 2 | 2025-03-21 发布；稳定 2.2.1 2026-03-02 | Wikipedia |
| NNSVS | 最后提交 2023-10-09 | GitHub API（缓存） |
| SOME / GAME | SOME 2024；GAME 2026-08 更新 | 仓库 README |
| TIFA | v1.0.0 模型；仓库 2026-10 活跃 | README |
| MFA | 2026-10-08 更新 | GitHub API（缓存） |

## 附录 B：证据文件（本目录下）

@@@
_work/
  gh/*.json                 GitHub 仓库元数据缓存（license/stars/pushed_at）
  readme/*.md               各项目 README/文档快照（DiffSinger、OpenUtau、ACE-Step、YuE、SoulX、YingMusic、DiffRhythm、SongGen、RVC、Seed-VC、Amphion、so-vits-svc、SOME、GAME、TIFA、g2pflow、MakeDiffSinger 等）
  readme/soulx_music_json.json   SoulX 元数据字段实测
  repos/                    浅克隆：DiffSinger、OpenUtau、MakeDiffSinger、svs-index、SongGen、DiffRhythm
  proto_midi2ds.py          旋律+拼音→.ds 原型（仅标准库，自检通过）
  out_demo.ds               原型输出
  analyze_svs_index.py / inspect2.py / inspect_svs_index.py  社区声库索引统计（107 singers / 318 variants / 14 个 DiffSinger）
  pylibs/                   仅 pypinyin 的 --target 安装（多音字实测）
@@@

## 附录 C：主要参考 URL

- DiffSinger：https://github.com/OpenVPI/DiffSinger ｜ https://diffsinger.com ｜ https://github.com/openvpi/MakeDiffSinger ｜ https://github.com/openvpi/SOME ｜ https://github.com/openvpi/GAME ｜ https://github.com/openvpi/TIFA ｜ https://github.com/openvpi/g2pflow ｜ https://github.com/openvpi/vocoders ｜ https://github.com/openvpi/dataset-tools
- 编辑器：https://github.com/openutau/OpenUtau ｜ https://github.com/openutau/OpenUtau/issues/1615 ｜ https://github.com/diffscope/diffscope-project ｜ https://openvpi.github.io/（OpenSVIP 工程格式转换）
- 零样本/改词：https://github.com/Soul-AILab/SoulX-Singer ｜ https://arxiv.org/abs/2602.07803 ｜ https://huggingface.co/Soul-AILab/SoulX-Singer ｜ https://github.com/ASLP-lab/YingMusic-Singer-Plus ｜ https://arxiv.org/abs/2603.24589 ｜ https://huggingface.co/ASLP-lab/YingMusic-Singer-Plus
- 全曲：https://github.com/ace-step/ACE-Step-1.5 ｜ https://arxiv.org/abs/2602.00744 ｜ https://huggingface.co/ACE-Step/Ace-Step1.5 ｜ https://github.com/multimodal-art-projection/YuE ｜ https://arxiv.org/abs/2609.33757 ｜ https://huggingface.co/m-a-p/YuE2-3B ｜ https://huggingface.co/m-a-p/SheetSage2 ｜ https://github.com/ASLP-lab/DiffRhythm ｜ https://github.com/LiuZH-19/SongGen
- 转换/工具：https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI ｜ https://github.com/Plachtaa/seed-vc ｜ https://github.com/svc-develop-team/so-vits-svc ｜ https://github.com/open-mmlab/Amphion ｜ https://github.com/spotify/basic-pitch ｜ https://github.com/yxlllc/RMVPE ｜ https://github.com/m-bain/whisperX ｜ https://github.com/MontrealCorpusTools/Montreal-Forced-Aligner ｜ https://github.com/mozillazg/python-pinyin
- 社区索引：https://openutau.github.io/svs-index/ ｜ https://github.com/openutau/svs-index
- 商业 UX 参照：https://dreamtonics.com/synthesizerv/ ｜ https://www.vocaloid.com/en/ ｜ https://acestudio.ai/ ｜ https://en.wikipedia.org/wiki/Synthesizer_V
