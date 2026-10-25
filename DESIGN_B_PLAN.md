# AMS Studio B 方案完整设计与实施计划 v2

> 版本：v2（2026-10-24）  
> 本文件修订 v1 计划中的三处理解偏差：歌曲换源、变声器、MSVC/index。  
> 原则：先统一概念，再设计架构；所有由程序管理的路径都从安装根 data 推导；不声称未验证的能力，不伪造 RVC index。

---

## 0. 先纠正三处概念

### 0.1 歌曲换源

正确语义：

- 输入：**已经分离好的纯人声**。
- 处理：用目标音源 pth 做 RVC 换声。
- 输出：**纯人声**，目标音色。
- 不做：不重新配伴奏，不与伴奏混合，不导出成品歌曲。
- 伴奏混音属于用户的 DAW/剪辑工程，不属于 AMS 的歌曲换源功能。

因此：

- 当前插件页里的“变声器”其实是**离线文件换声**，功能上就是歌曲换源。
- 它应该改名为“歌曲换源”，输入纯人声，输出纯人声。
- 不再使用 .msvc 文件。

### 0.2 变声器

正确语义：

- 实时处理麦克风或任意音频输入；
- 输出一路可以被其他软件当作“麦克风/音频输入”的设备；
- 类似心月变声器的实时变声；
- 需要 pth 音源；index 可选；
- 不是“选择一个文件、输出一个 wav”的离线功能。

因此：

- 当前“变声器”页签不是真正的变声器，应该迁移为歌曲换源。
- 真正的实时变声器需要一个独立的本地音频服务 + 虚拟音频设备；
- 不能只靠浏览器/插件页面做实时音频 I/O。

### 0.3 MSVC

正确语义：

- MSVC 只是心月格式的**加密模型容器**；
- 项目正常换声不需要 MSVC；
- 唯一用途是把用户自己的 .msvc 转成标准 RVC pth；
- 不参与歌曲换源、实时变声、训练的主流程。

---

## 1. RVC index 是什么、是否必须

### 1.1 index 的作用

RVC 的 index 是一个 FAISS 近似最近邻索引，内容不是模型权重，而是训练数据经过 HuBERT 提取出来的特征向量。

推理时：

- 模型本身负责把输入音频转成目标音色；
- index 负责从训练集特征中检索“最接近的目标特征”，在推理中做特征混合；
- 参数 index_rate 控制混合比例，0 表示完全不用 index。

它主要改善：

- 目标音色相似度；
- 减少音色泄漏；
- 让转换结果更接近训练集的声音。

它不负责：

- 不是 pth 的必需附件；
- 不改变模型结构；
- 不是音频文件；
- 不是 MSVC 容器的一部分。

### 1.2 index 是否必须

不是必须。

- 没有 index，RVC 仍可推理；index_rate 设为 0 即可。
- 有匹配 index 时通常效果更好；但 index 必须和同一个 pth/训练集匹配。
- 用另一个模型的 index 不会更好，可能更差。

### 1.3 为什么 MSVC 不能生成 index

index 的原料是训练音频的 HuBERT 特征（通常是 3_feature768 目录下的大量 npy 文件），不是 pth 权重。

MSVC 容器里只有模型权重，没有原始训练音频和特征。  
仅凭 pth 无法反推出训练集的 HuBERT 特征分布，因此无法生成正确的 index。

可行的三种情况：

1. 用户另外有同名 index：直接放入 data/voices/名称/ 使用。
2. 用户有原始训练数据集：可以用 RVC 训练/索引工具重新生成 index。
3. 都没有：使用 index_rate=0，模型仍可用，只是没有检索增强。

结论：MSVC 转 pth 只能生成 pth；不能承诺生成 index，也不应该伪造 index。

### 1.4 训练时如何生成 index

RVC20260311 的训练链路包含：

- 预处理音频；
- 提取 HuBERT 特征到 3_feature768；
- 训练模型；
- 用特征文件构建 FAISS index；
- 把 pth 和 index 一起注册到音源库。

所以训练产出的模型可以同时有 pth 和 index；MSVC 转换是另一条路，不能混为一谈。

---

## 2. 变声器（实时）架构

### 2.1 现实约束

Windows 上没有任何普通用户态程序可以凭空创建一个新的录音设备。  
其他软件要看到“AMS 麦克风”，需要一个**虚拟音频设备驱动**。

可选方案：

- VB-Cable：最常用，安装后出现 CABLE Input / CABLE Output；
- VoiceMeeter：功能更多，配置复杂；
- 用户已有的其他虚拟声卡。

AMS 不应静默安装驱动。建议：

- 检测已安装的虚拟音频设备；
- 如果没有，给出官方安装说明和检测按钮；
- 用户安装后，在 AMS 里选择输出到 CABLE Input；
- 在 OBS/YY/Discord/游戏里选择 CABLE Output 作为麦克风。

### 2.2 实时服务

新增独立本地服务：ams-vc-service。

职责：

- 枚举 WASAPI 输入/输出设备；
- 采集麦克风、线路输入，或可选的系统音频 loopback；
- 调用 RVC20260311 的实时引擎 rtrvc；
- 输出到虚拟音频设备或物理监听设备；
- 提供 start/stop、设备选择、音源热切换、参数调整、电平表、延迟统计；
- 通过 127.0.0.1 + token 的本地 API 受控；
- 与 Gateway 共享 GPU 租约，避免和训练/分离打架。

不把实时音频放进浏览器/Electron 渲染进程，因为：

- 浏览器不能稳定访问原生低延迟设备；
- 无法处理 WASAPI 独占、ASIO、虚拟声卡；
- 关闭页面会中断音频。

### 2.3 页面

“变声器”页面变成实时控制台：

- 输入设备（麦克风/线路输入/可选的系统 loopback）
- 输出设备（虚拟声卡 / 物理设备）
- 监听设备（可选）
- 音源 pth
- index（可选）
- 变调、共振峰、index rate、f0 方法
- block time、crossfade、额外上下文
- 启动/停止
- 输入/输出电平表
- 当前延迟
- 热切换音源
- 错误提示：设备不存在、虚拟声卡未安装、GPU 不可用、音源不匹配

---

## 3. 歌曲换源（离线）架构

### 3.1 输入输出

- 输入：纯人声文件（wav/flac/mp3/m4a 等）。
- 目标：选择一个 RVC 音源 pth。
- 处理：convert.rvc。
- 输出：纯人声 wav（目标音色），不混伴奏。

### 3.2 页面

- 输入人声
- 目标音源下拉
- 参数：pitch/f0 方法/index rate/filter/protect/rms/resample
- 任务提交
- 产物列表
- 可选“从歌曲分离人声”按钮：调用 pymss，把 vocals 填回输入框；不自动混伴奏。

### 3.3 与 MSVC 的关系

- 无关。
- MSVC 转换独立放在“工具/MSVC 转换”，只在用户有 .msvc 模型时使用。
- 歌曲换源不出现 .msvc 字段。

---

## 4. B 方案的完整功能范围

B 方案按设计文档的 10 个插件 + 实时变声 + 当前已有功能设计。  
以下是目标产品范围，不代表当前已实现。

| 模块 | 核心能力 | 主要服务/任务 |
|---|---|---|
| 启动器 | 安装/环境/组件/内核/Gateway/音源/文件夹/介绍页 | Tauri commands |
| 内核 | DSH 工作区、插件宿主、AI 对话 | DSH |
| 音源炼制 | RVC 训练、数据集预处理、index 构建、注册音源 | train.rvc |
| 人声分离 | pymss stems | separate.pymss |
| 变声器 | 实时麦克风/音频输入 → 虚拟麦克风 | vc.session / ams-vc-service |
| 歌曲换源 | 纯人声 → 目标音色纯人声 | convert.rvc |
| TTS | Piper 文本转语音 | tts.synthesize |
| 数据集工作室 | 导入、切片、校验、清洗、元数据 | dataset.prepare |
| 音色库 | 扫描、试听、标签、index 管理、导入导出 | 读 voices |
| 混音母带 | 多轨、响度、限幅、导出（独立，不参与歌曲换源） | mix.master |
| 评测器 | 客观指标、A/B、报告 | evaluate.audio |
| 项目管理 | 工程、步骤、产物引用、打开目录、导出 | project.* |
| MSVC 工具 | .msvc → pth（不生成 index） | convert.msvc |

如果“混音母带”与“不做伴奏”冲突：可以保留为独立工具，但不进入歌曲换源链路；是否保留需要你确认。

---

## 5. 目标数据模型

### 5.1 目录

~~~text
data/
  projects/<project-id>/
    project.json
    vocals/
    stems/
    artifacts/
    exports/
  datasets/<dataset-id>/
    source/
    processed/
    metadata.json
  voices/<音源名>/
    <音源名>.pth
    <音源名>.index（可选）
    voice.json
  tts_voices/<模型名>/
    <模型名>.onnx
    <模型名>.onnx.json
  artifacts/<job-id>/
  recordings/vc/<session-id>/
  models/
  cache/
  backups/
  logs/
  runtime/
  components/
  tools/
  kernel/
  config/
  tmp/
~~~

启动器初始化时创建全部目录；不删除已有内容。

### 5.2 工程对象

project.json 记录：

- 工程 id/名称/创建时间；
- 输入素材与导入来源；
- 每一步的 job id、参数、产物路径；
- 使用的音源/模型版本；
- 导出记录。

所有引用使用 data 内相对路径，不写绝对盘符。

---

## 6. 服务与 API 规划

### 6.1 Gateway 任务

保留并扩展：

- ping
- convert.rvc
- separate.pymss
- convert.msvc
- tts.synthesize
- train.rvc（新增）
- dataset.prepare（新增）
- mix.master（新增，若保留混音）
- evaluate.audio（新增）

### 6.2 实时变声服务

不走 Job 队列，使用会话：

- POST /vc/sessions
- GET /vc/sessions/{id}
- POST /vc/sessions/{id}/stop
- GET /vc/devices
- POST /vc/sessions/{id}/switch-voice
- GET /vc/sessions/{id}/stats

由 Gateway 或独立控制服务代理；音频数据不经过浏览器。

### 6.3 文件夹与素材

- /ams/api/open-folder（白名单 kind）
- /ams/api/projects/import
- 播放/试听：通过受控 local file route，不暴露任意路径

---

## 7. 上游来源与介绍页

介绍页只对从开源项目搬来的模块写来源；AMS 自研的启动器、插件、Gateway、MSVC 转换器不强行编造上游。

| 模块 | 来源 | 版本/说明 |
|---|---|---|
| 内核 | DeepSeek Harness（DSH） | 0.2.1-rc.2，MIT；https://github.com/deepseek-ai/deepseek-harness |
| 插件框架 | Cordis | 随 DSH 使用的开源插件框架；https://github.com/cordiverse/cordis |
| RVC 推理/训练/实时 | RVC-Project Retrieval-based-Voice-Conversion-WebUI | 当前 runtime 基于 RVC20260311，约 2.2.231006 基线；MIT；https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI |
| pymss 分离 | pymss / pymss-core | 2.1.9 / 0.1.11，MIT；https://github.com/pymss-project/pymss |
| TTS 引擎 | Piper | 以实际二进制来源和版本为准；https://github.com/rhasspy/piper |
| 默认曼波音源 | 第三方音源资产 | 不是开源代码；介绍页必须标明来源/授权 |
| TTS 默认音色 | Piper 模型 | 标明模型来源/许可 |
| MSVC → pth | AMS 自研 msvc2pth 0.4.0 | 不写开源上游；说明是逆向互操作工具 |

介绍页只写实际使用版本，不把设计目标 2.3.260718 写成当前版本。

---

## 8. 参数设计

### 8.1 RVC（当前 runtime 支持范围）

推理参数：

- f0_up_key：-24..24
- f0_method：pm / harvest / crepe / rmvpe（以当前 runtime 代码为准，不写不存在的 fcpe）
- index_rate：0..1
- filter_radius：0..7
- resample_sr：0 或模型支持的采样率
- rms_mix_rate：0..1
- protect：0..0.5
- index_path：可选

实时参数：

- input/output device
- block_time
- crossfade_length
- extra_context
- monitor
- 输入增益
- 模型热切换

训练参数：

- 数据集
- 采样率 32k/40k/48k
- 是否带 f0
- f0 提取方法
- epochs
- batch size
- 保存间隔
- 是否保存最新
- GPU
- 版本 v2

### 8.2 pymss

以 pymss 2.1.9 为准：

- model
- device
- chunk_size
- overlap_size
- stem_batch_size
- normalize
- standardize
- stems
- output format

### 8.3 Piper

- model
- config
- speaker（多说话人模型）
- length_scale
- noise_scale
- noise_w
- sentence_silence
- 输出采样率

### 8.4 MSVC

- input
- out
- name
- reference pth
- no-cache

不包含 index 生成。

---

## 9. 分阶段实施

### 阶段 A：基础修复与概念纠正（1–2 周）

- RVC 目标音源默认/必填修复；
- 当前“变声器”离线页迁移为“歌曲换源”；
- 歌曲换源输入纯人声，输出纯人声，不混伴奏；
- MSVC 移出换声流程，独立工具；
- TTS 音源目录 data/tts_voices；
- data 目录初始化；
- 变声音源/TTS音源/任务产物三个打开文件夹按钮；
- 启动器介绍页骨架。

验收：现有 TTS、pymss、歌曲换源可用；无 401/参数缺失；data 目录完整。

### 阶段 B：实时变声器（2–4 周）

- ams-vc-service；
- RVC20260311 rtrvc 接入；
- 设备枚举、输入/输出/监听；
- 虚拟音频设备检测与 VB-Cable 引导；
- 实时参数、热切换、电平、延迟；
- 会话 API/UI；
- GPU 租约。

验收：麦克风输入 → 实时换声 → CABLE Output 可被 OBS/语音软件识别为麦克风；延迟达到目标。

### 阶段 C：参数与音源管理（1–2 周）

- RVC/pymss/TTS 全参数面板；
- 参数模板；
- 音色库高级管理；
- TTS 多音色；
- MSVC 工具 UI 和说明。

### 阶段 D：音源炼制与数据集工作室（2–4 周）

- train.rvc；
- 数据集导入/切片/校验；
- 特征提取/训练/index；
- 注册音源；
- 小样本验收。

### 阶段 E：歌曲换源完整化（1–2 周）

- 从分离结果一键填入；
- 长音频切片/接缝处理（如需要）；
- 质量检查、A/B、批量；
- 纯人声导出。

### 阶段 F：项目/混音/评测/填词（按插件逐个交付）

- 项目管理；
- 混音母带（独立，不进歌曲换源）；
- 评测器；
- V 家填词（若纳入 B）；
- 每个插件单独验收。

### 阶段 G：整合发布

- 介绍页补全；
- feed/setup；
- 干净 VM；
- 数据迁移与升级；
- 全链路回归。

---

## 10. 需要你确认的问题

1. **实时变声的虚拟设备**  
   你是否接受使用 VB-Cable 这类虚拟声卡（需要用户另行安装）？  
   还是要求 AMS 自带虚拟音频驱动？后者需要签名内核驱动，风险和工程量都很大。

2. **实时输入范围**  
   只支持麦克风/线路输入，还是也要支持“系统正在播放的声音”loopback？  
   后者需要额外 WASAPI loopback 支持。

3. **实时输出**  
   只输出到虚拟声卡，还是同时支持物理设备监听？

4. **B 方案是否包含混音母带**  
   你说“再次配伴奏不属于本软件”，那是否保留独立的混音母带插件？  
   建议保留为独立工具，但绝不进入歌曲换源链路。

5. **B 方案是否包含 V 家填词**  
   设计文档里有 V 家填词（DiffSinger 等），但不在 10 个插件列表中。  
   是否纳入本次 B？

6. **MSVC index**  
   你是否能接受“MSVC 只转 pth；index 需要另找或另有训练数据；没有就用 index_rate=0”？  
   如果你要求必须有 index，需要提供原始数据集或已有 index，无法从 pth 自动生成。

7. **歌曲换源输入**  
   是否严格要求输入为已分离纯人声；需要“从歌曲一键分离出人声”的辅助按钮吗（仍然不混伴奏）？

8. **实时变声音源**  
   实时变声也使用 data/voices 下的 pth；index 可选。  
   是否允许同时加载多个音源并热切换？

9. **训练验收数据**  
   你准备用多少分钟、什么语种/底噪的数据做首次训练验收？  
   这决定训练阶段能否按预期完成。

---

## 11. 当前不建议马上做的事

- 不要先在现有 UI 上继续加“歌曲换源”占位页；
- 不要把 MSVC 参数继续塞进歌曲换源；
- 不要在没有虚拟声卡方案前做实时变声 UI 的假接口；
- 不要先做训练 UI 而不验证 runtime 训练脚本；
- 不要在未确认 B 范围前一次性铺开所有插件。

---

## 12. 下一步

你确认第 10 节的问题后，我会把本计划整理成可执行的里程碑任务，并从阶段 A 开始实现；先修当前可用性，再进入实时变声和训练。
---

## 附录 A：B 方案模块详细设计

### A.1 音源炼制（train.rvc）

- 页面输入：数据集目录、模型名、采样率、是否带 f0、f0 方法、epochs、batch size、保存间隔、GPU。
- 数据集目录：data/datasets/<名称>，原始音频放在 source/，预处理产物放 processed/。
- 服务：使用 RVC20260311 的训练脚本与预训练模型；训练、特征提取、index 构建在独立子进程运行。
- 产物：data/voices/<模型名>/<模型名>.pth + <模型名>.index + voice.json。
- 任务阶段：prepare → extract_f0 / extract_feature → train → index → register。
- 取消：杀进程树；保留已完成的中间产物但标记为未完成，不自动注册。
- 验收：10–20 分钟小样本训练出可推理 pth；index 可被 RVC 加载；RVC 任务能用新音源。
- 风险：训练时间、显存、数据质量；必须先做 runtime 脚本 smoke test。

### A.2 人声分离（separate.pymss）

- 页面输入：整曲、模型、device、高级参数。
- 数据：输入复制到 data/projects/imports；产物 data/artifacts/<job>/stems。
- 服务：pymss 2.1.9。
- 输出：vocals/drums/bass/other 等 stems + separation-manifest.json。
- 与歌曲换源联动：选择 vocals 结果填入歌曲换源输入；不自动混伴奏。
- 验收：不同时长、采样率、声道输入；取消；GPU/CPU。
- 参数：model/device/chunk_size/overlap_size/stem_batch_size/normalize/standardize/stems。

### A.3 变声器（实时，vc.session）

- 页面：输入设备、输出设备、监听设备、音源、index、pitch、formant、index_rate、f0_method、block_time、crossfade、extra_context。
- 服务：ams-vc-service，独立进程；RVC20260311 rtrvc。
- 输入：WASAPI 麦克风/线路输入；可选系统 loopback。
- 输出：虚拟音频设备（VB-Cable 等）或物理设备；可选本地监听。
- 会话 API：start/stop/status/switch-voice/stats。
- 设备：检测虚拟声卡，不存在时引导安装，不静默装驱动。
- 验收：OBS/语音软件把 CABLE Output 识别为麦克风；延迟目标 p95 ≤ 300 ms；热切换不中断。
- 风险：虚拟声卡驱动、设备独占、GPU 占用、回声/监听。

### A.4 歌曲换源（convert.rvc）

- 页面：纯人声输入、目标音源、参数、任务列表。
- 不做：伴奏输入、混音、成品导出。
- 产物：纯人声 wav + sidecar json。
- 可选辅助：从人声分离结果填路径；不自动混伴奏。
- 验收：4 分钟人声可完成（当前 runtime 整段或切片）；输出时长与输入一致。

### A.5 TTS（tts.synthesize）

- 音源目录：data/tts_voices/<模型名>/*.onnx + *.onnx.json。
- 模型来源：runtime 内置 + 用户目录；两者合并扫描。
- 参数：model、speaker、length_scale、noise_scale、noise_w、sentence_silence。
- 输出：data/artifacts/<job>/tts.wav。
- 验收：默认模型和用户模型都能合成；无效模型给出明确错误。

### A.6 数据集工作室（dataset.prepare）

- 功能：导入、试听、切片、裁剪、重采样、去静音、响度归一、重复/损坏检测、元数据。
- 数据：data/datasets/<id>/source、processed、metadata.json。
- 输出：训练可用的音频目录 + 报表。
- 验收：10–20 分钟样本可清洗到训练规范；不修改原始文件。
- 风险：音频处理依赖 ffmpeg/librosa；长文件内存。

### A.7 音色库（voices）

- 功能：扫描、试听、标签、备注、index 管理、导入/导出、缺失 index 提示、文件 hash。
- 数据：data/voices/<名称>/<名称>.pth(.index)(voice.json)。
- 验收：非法目录不被误识别；删改前确认；试听走本地受控路由。
- 不负责：训练、TTS 音源；TTS 在 data/tts_voices。

### A.8 混音母带（mix.master）

- 用户已明确：歌曲换源不负责配伴奏。因此混音母带只能作为独立工具，不应自动出现在歌曲换源流程。
- 功能（若保留）：多轨输入、音量/声像、EQ/压缩（最小 DSP）、响度归一、限幅、导出。
- 服务：ffmpeg + numpy/pyloudnorm；data/projects/<id>/mix。
- 验收：输入/输出轨数正确；响度指标；不改变原轨。
- 是否纳入 B 需要确认（见主问题 4）。

### A.9 评测器（evaluate.audio）

- 功能：时长/采样率/声道、削波、静音、LUFS、SNR、音高偏差、可选的相似度指标。
- 产物：报告 JSON + 摘要；不自动删除产物。
- 验收：已知样本指标可复现；错误输入给明确原因。
- 风险：主观音质不能只靠指标；需 A/B 人工确认。

### A.10 项目管理（project）

- 功能：新建/打开/删除工程、素材引用、步骤记录、产物关联、打开目录、导出包。
- 数据：data/projects/<id>/project.json + 子目录。
- 不做：自动混伴奏；只管理工程与步骤。
- 验收：每一步 job 的参数/产物可追溯；路径全部 data 内相对路径。
- 风险：数据迁移/旧版本兼容。

### A.11 MSVC 工具（convert.msvc）

- 页面：输入 .msvc、输出目录、名称、参考 pth、no-cache。
- 输出：pth + conversion.json；不生成 index。
- 说明：MSVC 容器无检索特征；没有 index 时使用 index_rate=0。
- 验收：已知 .msvc 转换后能被 RVC 加载；无 index 时 RVC 仍能推理。
- 风险：参考模型/边界校准；新容器变体；许可证/来源。

### A.12 V 家填词（可选）

- 是否纳入 B 需要确认。
- 若纳入，按设计文档 DiffSinger/OpenUtau 路线做独立 P1 里程碑，不与 RVC/TTS 混为一个 Job。
- 不应在本期承诺“完整歌手合成”而没有模型许可与工程验证。

---

## 附录 B：里程碑与验收节奏

| 里程碑 | 内容 | 依赖 | 验收产物 |
|---|---|---|---|
| M0 | 概念冻结、目录契约、上游版本表 | 用户确认 | 本设计定稿 |
| M1 | 现有功能修复 + 歌曲换源改名 + MSVC 独立 + data 初始化 + 文件夹按钮 + 介绍页骨架 | M0 | 当前安装可用的更新包 |
| M2 | 实时变声服务 MVP | M1、虚拟声卡方案 | 麦克风→虚拟麦克风可听 |
| M3 | 参数面板 + TTS 多音源 + 音色库 | M1 | 参数/音源可用 |
| M4 | 训练 + 数据集工作室 | M1、RVC20260311 验证 | 小样本训练出 pth+index |
| M5 | 歌曲换源完整化 + 质量/A-B | M1/M3 | 纯人声批处理 |
| M6 | 项目/混音/评测/填词（按确认范围） | M1–M5 | 逐插件验收 |
| M7 | 发布/干净 VM/升级 | 全部 | feed/setup/说明书 |

验收原则：

- 每个里程碑必须先通过“真实任务”验收，不以 UI 完成代替后端完成。
- 任何“未实现”必须在介绍页和 UI 明确标记。
- 所有数据目录变更必须可重复执行且不删除用户文件。
- 所有新增 job 必须记录参数、输入、输出、错误和可取消性。

---

## 附录 C：测试矩阵

### 数据与 UI

- 全新安装 data 目录自动补齐。
- 已有 data 升级不丢 voices/projects/tts_voices。
- 三个打开文件夹按钮指向正确目录。
- 介绍页各章节可打开，返回启动器。
- UI 不出现绝对盘符。

### 音频任务

- TTS：默认模型、用户模型、长文本、超长文本错误。
- 分离：wav/mp3/flac、单/双声道、CPU/GPU、取消。
- 歌曲换源：短/长纯人声、不同音源、index 有/无、pitch 边界。
- MSVC：已知样本、无 index、错误 reference、非法文件。
- 训练：小样本、取消、失败恢复、index 构建、音源注册。
- 实时：设备切换、虚拟声卡、热切换、延迟、断连恢复。

### 发布

- 签名 feed 验证。
- 新 setup 安装。
- 旧版升级。
- 干净 Windows VM 从零安装。
- 无 C 盘 AMS 数据、无持久环境变量。

---

## 附录 D：需要提前验证的技术点

1. RVC20260311 的实时引擎在目标机器上能用 sounddevice 正常打开设备。
2. VB-Cable 的 CABLE Input/Output 命名和默认格式。
3. RVC 训练脚本在当前 runtime 的 Python 3.9 + torch 2.0/cu118 下能跑通。
4. index 构建脚本与 3_feature768 路径。
5. pymss 2.1.9 的 inference_params 与模型目录。
6. Piper 多说话人模型和参数兼容性。
7. DSH 插件页面能否容纳实时控制台的大量状态更新而不卡。
8. 长音频 RVC 是否需要切片；当前 runtime 的显存峰值。
9. 文件选择/试听在 WebView 中的本地受控访问。
10. 所有打开文件夹/路径 API 的目录逃逸防护。
