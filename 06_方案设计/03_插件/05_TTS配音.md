# 插件 05：TTS / 配音（ams.music.tts）详细方案

> 版本：v1.0（2026-10-19）【决策】【待定】【风险】标注见各节｜作者：gptsovits-analyst｜状态：方案（未开发）
> 上位文档：`06_方案设计/00_总体设计与约定.md`（下称「00」；本文所有约定以 00 为准，冲突处以 00 为准并在文中标【偏差】）
> 输入依据：`02_ai配音_GPT-SoVITS/报告.md`（本机 GPT-SoVITS v2Pro 实测）、`05_新方案探讨/03_开源栈调研/报告.md`（2026-10 TTS 选型/许可）、`05_新方案探讨/可行性探讨.md`、`05_新方案探讨/02_SVS填词/报告.md`
> 关联分册：`07_音色库.md`（声库管理）、`01_启动器`（组件/模型分发）、`06_数据集工作室`（训练数据）、`08_混音母带`（成品对轨）、`03_变声器`、`04_歌曲换源`（音色跨用途关联）

---

## 1. 目标与非目标

### 1.1 目标

| # | 目标 | 说明 |
|---|---|---|
| G1 | **中文角色配音** | 文本 → 指定角色音色的语音（TTS），成品可直接进混音/剪片/游戏 Mod 工作流 |
| G2 | **主后端 = GPT-SoVITS v2Pro** | 微调（1–5 分钟数据）+ 推理，中文最实用、MIT 许可、生态成熟；本机已有可用样本（Mualani v2Pro） |
| G3 | **快捷后端 = Qwen3-TTS** | Apache-2.0，3 秒参考音频零样本克隆、原生流式；用于"先听效果/临时配音/无训练数据"场景 |
| G4 | **语气可控** | 通过"同一角色的多套参考音频（平静/生气/开心/疲惫…）"实现语气切换；参数侧提供语速/停顿/采样参数 |
| G5 | **批量与时间线** | 剧本级批量合成（CSV/JSON 脚本、多角色）；P1 提供片段时间线并可导出带时间码的 WAV/SRT |
| G6 | **音色资产化** | 训练/导入的 TTS 音色统一落到 `AMS_HOME/tts_voices/`，与 RVC 音源、SVS 声库按"同一角色"关联 |
| G7 | **AI 可编排** | 全部能力经 Gateway 任务 + `music_tts_*` 工具暴露，AI 可以"拆剧本→分配角色→批量生成→校验→导出" |
| G8 | **合规可交付** | 引擎/权重许可清晰、AI 生成标识、声音来源授权记录、许可红线硬拦截 |

### 1.2 非目标（一期明确不做）

| 不做 | 原因/归属 |
|---|---|
| 唱歌/歌声合成（SVS） | 由 P1 `插件 04/填词成歌` + DiffSinger 路线负责（见 02_SVS填词）；TTS 只做"说" |
| 语音/歌声**转换**（变声、换源） | 归 `插件 03 变声器`、`插件 04 歌曲换源`（RVC） |
| 声音设计（VoiceDesign：用文字描述"造一个声音"） | Qwen3-TTS-VoiceDesign 能力存在，但一期不做 UI；登记为 P2【待定】 |
| 云端 TTS / 大模型在线合成 | 00 第 1 节「本地推理不依赖云端」；AI（DeepSeek）只做规划/文案/校验 |
| 声音克隆的法律判断 | 应用只做"记录来源与授权"，不替用户判断合法性（00 第 1 节 合规姿态） |
| 内置任何音色/权重 | 00 第 1 节：应用不分发音源/模型权重 |

---

## 2. 用户与场景

### 2.1 用户画像

| 用户 | 需求 | 关键路径 |
|---|---|---|
| 同人/二创作者 | 给角色配台词、做剧情向短视频 | 导入角色音色 → 粘贴台词 → 选语气 → 批量生成 → 导出 |
| 游戏 Mod / 整活作者 | 大量短句、需要固定音色与语气 | 剧本 CSV → 多角色批量 → 时间码文件 |
| 有声内容创作者 | 长文本、旁白/对话分角色、质量稳定 | 长文分句 → 单角色长文本 → 批量试听/重生成 |
| 音源炼制玩家 | 已有 RVC 音源，想同角色做 TTS | 用同批数据训练 GPT-SoVITS → 与 RVC 音源关联 |
| AI 操作者（Agent） | 用自然语言驱动全流程 | `music_tts_*` 工具 + `ams-配音工作流` skill |

### 2.2 典型场景（验收导向）

| 场景 | 输入 | 输出 | 验收要点 |
|---|---|---|---|
| S1 单句试听 | 一句台词 + 角色 + 语气 | 1 个 wav（32k 或 48k）+ 时长/RTF | 首次试听 ≤ 3 分钟（含模型加载）或已有缓存 ≤ 20 s |
| S2 剧本批量 | 20 句 CSV（角色/语气/文本） | 20 个 wav + manifest.csv | 逐句进度可见、可取消、失败可单句重试 |
| S3 长文旁白 | 3000 字中文 | 分段 wav + 合并 wav | 自动按句切分（≤1000 字/段）、无漏字、停顿自然 |
| S4 训练角色 | 3 分钟干声 + 校对文本 | tts_voices/<角色>/ + 训练指标 | 训练可断点续训、产出可试听、指标达标 |
| S5 零样本快配 | 6 s 参考音频 + 一句台词 | wav（Qwen3-TTS） | 10 秒级出首包（GPU），音色可辨 |
| S6 AI 全流程 | "把这段剧本用 XX 音色配成 3 个角色" | 工程 tts 片段 + 成品 | AI 自动拆句/分配/生成/校验并汇总报告 |

---

## 3. 功能清单（P0/P1/P2 + 验收标准）

### 3.1 功能表

| ID | 功能 | 优先级 | 验收标准 |
|---|---|---|---|
| F-01 | GPT-SoVITS v2Pro 推理（单段） | **P0** | 输入 ≤1000 字中文 → 输出 32 kHz 单声道 wav；片段参数（语速/停顿/切句法/top_k/top_p/温度）生效；输出电平正常（峰值 > -6 dBFS、无削波） |
| F-02 | 音色选择（tts_voices） | **P0** | 列表/grid 可选；支持"多参考音频=多语气"切换；未选音色时禁止生成并提示 |
| F-03 | 参考音频临时覆盖 | **P0** | 允许本次任务临时用另一段 3–10 s 音频+文本（不入库）；非法时长直接拦截并提示 |
| F-04 | 脚本批量合成 | **P0** | CSV/JSON 脚本（角色/语气/文本/可选参数）→ 逐段产物 + manifest；进度/取消/单段重试；失败不中断整批（可配置） |
| F-05 | 文本预处理与校验 | **P0** | 多音字/数字/英文缩写/生僻字/超长/标点/空行检查；给出"可直接生成/建议修改"两类结果；不联网 |
| F-06 | 试听与 A/B | **P0** | 波形+播放器；同文本可保存多组参数快照对比；一键"再来一次"（换 seed/温度） |
| F-07 | 导出 | **P0** | wav（32k/48k/44.1k 可选重采样）、flac、mp3；命名模板；写入 AI 生成标识元数据（可选） |
| F-08 | 任务中心集成 | **P0** | 全部生成为 Job，可在「任务 Jobs」页查看/取消/重试；GPU 锁排队有原因提示 |
| F-09 | 训练/微调（GPT-SoVITS s1+s2） | **P0** | 1–5 分钟数据可训；自动 ASR 打标/校对；断点续训；产出落 tts_voices/ 并自动选参考音频；训练后自动评测 |
| F-10 | 与 RVC 音源关联 | **P0** | tts.json 记录 link.rvc_voice；UI 显示"同角色 RVC 音源：XX"；双向跳转 |
| F-11 | Qwen3-TTS 零样本 | **P1** | 3 s+ 参考音频 → 合成；10 语种；中文可辨；首次加载 ≤ 60 s（GPU） |
| F-12 | Qwen3-TTS 流式（P1） | **P1** | 首包 ≤ 800 ms（GPU，段首 ≤ 200 字）；SSE 增量音频可播 |
| F-13 | 时间线（片段对齐） | **P1** | 多片段拖拽排序/静音间隔/口型参考；导出时间码 JSON + SRT + 对齐后的合并 wav |
| F-14 | 项目集成 | **P1** | 生成结果落 `projects/<id>/tts/`，project.json 记录输入/参数/模型版本/产物 hash |
| F-15 | 生成后自动校验（ASR 回译） | **P1** | faster-whisper 回译 CER 计算；CER > 8% 标黄、> 15% 标红；可批量复检 |
| F-16 | 可选后端 IndexTTS-2.5 | **P2** | 默认不下发；用户自行安装并显式接受许可后可用；情感控制更强（实验室开关） |
| F-17 | 可选后端 CosyVoice3 | **P2** | Apache-2.0；低延迟流式/多语种；作为流式备选 |
| F-18 | VoiceDesign / 多说话人对话 | **P2** | 文字描述造声（Qwen3-TTS-VoiceDesign）；对话式多角色连读 |

### 3.2 验收基线（全局）

| 指标 | 目标值 | 说明 |
|---|---|---|
| 合成成功率 | ≥ 99%（排除用户非法输入） | 失败必须给可读原因 + 错误码 |
| CPU 端到端 RTF | ≤ 4.0（i9-13900HX 级，v2Pro） | 本机实测：20 字 → 4.24 s 音频用 16.2 s，RTF ≈ 3.8【事实】 |
| GPU RTF（RTX 4080 Laptop 16 GB） | ≤ 0.5（v2Pro，批量） | 由 03 调研推算（v2Pro 显存 3–5 GB、RTF < 1）【推断】 |
| 首次冷启动（加载模型） | ≤ 90 s（GPU）/ ≤ 120 s（CPU） | 需要预热与缓存策略，见 §9 |
| 长文本稳定性 | 3000 字无漏字、无中途崩溃 | 分句 + 每段独立合成 + 合并 |

---

## 4. 信息架构与页面/交互

### 4.1 页面与导航

| 页面 | 路由 key | 槽位 | 说明 |
|---|---|---|---|
| 配音工作台 | `tts` | `main` 整页 + `sidebar.panellist`「配音」 | 主工作区（见 4.2） |
| 批量任务面板 | `tts.batch` | 工作台内右侧抽屉 | 脚本导入/队列/进度/失败重试 |
| 训练向导 | `tts.train` | 工作台内模态（分步） | 数据→校验→训练→评测→入库 |
| 引擎/许可设置 | `settings.section` | 设置页分节「配音引擎」 | 后端开关、模型状态、许可确认 |
| 工具卡片 | `tool.call.toolview` | 对话内工具卡片 | 显示生成结果/进度/试听 |

### 4.2 配音工作台布局

~~~text
┌──────────────────────────────────────────────────────────────────────────┐
│ 顶栏：工程选择 | 角色选择器(核心组件) | 语气下拉 | 引擎徽标 | 运行状态      │
├───────────────────────┬─────────────────────────────────┬────────────────┤
│ 左：脚本编辑器         │ 中：试听/波形区                  │ 右：参数面板    │
│ - 行号 + 角色标签      │ - 播放器(波形/频谱)              │ - 引擎(自动/强制)│
│ - 语法高亮 [角色:语气] │ - 片段级波形 + 时长/RTF          │ - 切句法 cut0-5 │
│ - 导入 CSV/JSON/纯文本 │ - A/B 槽位（≥3 组参数快照）       │ - 语速/停顿     │
│ - 实时文本校验标记     │ - 导出当前/全部                  │ - top_k/p/T/种子│
│ - 段落/角色分组        │ - 参考音频试听（语气来源）        │ - 输出格式/重采样│
├───────────────────────┴─────────────────────────────────┴────────────────┤
│ 底栏：生成按钮 | 批量生成 | 取消 | 进度(段/总) | 预计剩余 | 日志抽屉        │
└──────────────────────────────────────────────────────────────────────────┘
~~~

**交互要点**

| 交互 | 行为 |
|---|---|
| 选中文本 + 指定角色 | 「生成选区」只合成高亮行，便于逐句迭代 |
| 双击片段 | 打开片段详情（参数快照、seed、重生成、复制参数到全篇） |
| 语气切换 | 立即显示该参考音频的试听波形与文本（语气=参考音频+文本，来源透明） |
| 超长文本 | 自动分段预览（按 cut3 中文句号）；提示"分 7 段，预计 3 分 20 秒" |
| 生成中 | 逐段回传结果（先听到第 1 段）；可取消；可"仅重生成失败段" |
| 失败 | 行内红标 + 原因（模型未装/显存不足/文本非法/参考音频非法）+ 修复按钮 |
| 导出 | 单段/全部/合并/按角色分轨；P1 带时间码 |

### 4.3 状态与空态

| 状态 | 展示 |
|---|---|
| 无引擎/模型未装 | 空态卡片：「GPT-SoVITS 未安装」→ 打开启动器组件中心安装（带体积/许可） |
| 无 tts_voices | 空态：「还没有 TTS 音色」→ 训练向导 / 导入 v2Pro 模型 / 用 Qwen3-TTS 零样本 |
| GPU 被占用 | 顶栏黄条「GPU 正被『歌曲换源』使用，已排队（预计 3 分钟）」 |
| 许可未确认 | 首次使用某后端弹许可确认（黄色级后端强制每次版本变更后确认） |

---

## 5. 技术架构与依赖

### 5.1 进程与调用链（遵循 00 第 3 节）

~~~text
Client 插件(React) ──HTTP(token)──► Gateway(/api/v1) ──┬─► ams-svc-gptsovits (Python 进程)
                                                      ├─► ams-svc-qwen3tts  (Python 进程)
                                                      ├─► ams-svc-indextts  (可选)
                                                      └─► ams-svc-cosyvoice3(可选)
Host 插件(Node) ── 注册 tools(music_tts_*) + skills + SSE 订阅 ─┘
~~~

- **不直接 import 引擎**：每个后端一个独立进程（00 第 3 节「后端服务进程各自独立运行时」），Gateway 负责拉起/健康检查/GPU 锁/日志。
- **进程内并发**：v2Pro 为单例模型 + 串行推理；一个服务进程内只跑 1 个生成任务（多实例由 Gateway 按显存排队决定，一期 1 实例）。
- **预热**：服务启动即加载模型（可配置 lazy）；启动器/网关记录"上次可用"状态，避免每次冷启动 90 s。

### 5.2 后端选型与版本/许可（精确到项目+版本+许可）

| 后端 ID | 项目/版本 | 引擎许可 | 权重许可 | 语言 | 克隆 | 流式 | 微调 | 一期定位 |
|---|---|---|---|---|---|---|---|---|
| `gptsovits` | GPT-SoVITS **tag 20250606v2pro**（commit d7c2210） | **MIT** | 底模随项目 MIT；用户音色模型版权归用户 | 中英日韩粤 | 5 s 零样本 / 1 min 微调 | 否（段级） | ✅ s1+s2 | **主力后端（P0）** |
| `gptsovits-v5` | 分支 `cuda_graph_accel_v5`（2026-10，未并入 release） | MIT | 同上 | 同上 | 同上 | 否 | ✅ | 实验开关（P2）【待定】 |
| `qwen3tts` | Qwen3-TTS-12Hz-**1.7B/0.6B-Base**（HF 2026-01-21） | **Apache-2.0** | **Apache-2.0** | 10 语种 | **3 s** | ✅ 双轨流式 | ✅ Base 可微调 | **快捷后端（P1）** |
| `qwen3tts-cv` | Qwen3-TTS-**CustomVoice**（9 预置音色+指令风格） | Apache-2.0 | Apache-2.0 | 10 语种 | 预置音色 | ✅ | — | P2（风格控制） |
| `cosyvoice3` | Fun-CosyVoice**3-0.5B-2512** | Apache-2.0 | Apache-2.0 | 10 语种 | ✅ | ✅ ~150 ms | ✅ | P2（流式备选） |
| `indextts` | IndexTTS-**2.5**（2026-08-10） | **bilibili 自定义许可** | 同左 | 中英日西阿 | ✅ | 一般（AR） | 社区微调 | **P2 实验室（黄色级，默认不下发）** |

共同依赖（各服务环境内独立安装，见 00 第 5 节）：

| 组件 | 版本 | 许可 | 用途 |
|---|---|---|---|
| Python | 3.10（gptsovits 服务锁 3.9.25 亦可，按 00 独立运行时） | PSF | 运行时 |
| PyTorch / torchaudio | 2.4+（CUDA 12.4）/ 2.6.0（CPU 降级） | BSD-3-Clause | 推理/训练 |
| transformers | ≥4.43, ≤4.50（v2Pro 约束） | Apache-2.0 | BERT/HuBERT |
| onnxruntime(-gpu) | 1.19.2 | MIT | G2PW 多音字 |
| faster-whisper | 1.2.1 | MIT | ASR 打标 / 回译校验 |
| cn2an / jieba / pypinyin | 0.5.24 / 0.42.1 / 0.55.0 | MIT | 中文前端 |
| faiss-cpu | 1.7.2 / 1.8+ | MIT | 仅 RVC 侧（TTS 不用） |
| pyloudnorm | 0.2.0 | MIT | 响度一致性 |
| DNSMOS P.835 / UTMOS22 | 模型权重 | 待核实（P1 评测用） | 质量打分 |

> 版本策略：**引擎锁 tag + 服务镜像锁依赖**（00 第 5 节「service.<id> 独立 SemVer」）；v2Pro 的 S1/S2 结构、ct2 权重格式、API 变更都就近受控；升级必须过 §11 回归集。

### 5.3 引擎能力边界（来自本机实测，必须写进 UI 提示）

| 约束 | 事实来源 | 产品处理 |
|---|---|---|
| 参考音频 **3–10 s**，超范围引擎直接抛错 | `02 报告` §6（TTS.py 校验） | 导入/选择时前置校验；时长不合规给"裁剪建议" |
| 输出 **32 kHz / 单声道 / PCM16**（v2Pro） | `02 报告` §7 | 导出可选重采样到 48k/44.1k（异步） |
| 引擎强依赖 **cwd=引擎根** 才能 import | `02 报告` §11.4 | 服务进程用固定工作目录启动，封装入口；禁止宿主直接 import |
| v2Pro 的 SoVITS `.pth` 文件头为 `b"05"` | `02 报告` §9.1 | 服务内用官方 `load_sovits_new` 逻辑读取，禁止直接 torch.load |
| 单例 + 串行 + 无 seed 暴露（原壳） | `02 报告` §5.7 | AMS 侧自行加 seed/fragment 参数并做任务队列 |
| 文本 ≤1000 字/段（工程经验值） | 本机壳限制 | 自动分段，UI 显示段数/预计时长 |

### 5.4 与 00 的偏差/补充（需回填 00 的登记表）

| 项 | 说明 | 处理 |
|---|---|---|
| 新增 Job 类型 `tts.batch`、`tts.eval` | 00 第 8 节 列举未含 | 【决策】新增并在 Gateway 任务注册表登记；命名遵循 00 的 `域.动作` |
| `tts_voices/` 内部格式 | 00 第 4 节 只规定目录名 | 【决策】本文 §6.1 定义 `tts.json` 为必需清单（因非 RVC 标准格式） |
| 接口新增 `/tts/*` | 00 第 8 节 仅示例 | 【决策】按本文 §7 扩展，全部挂 `/api/v1` |

---

## 6. 数据模型与目录

### 6.1 `tts_voices/` 音色包格式（本文定义，P0）

~~~text
AMS_HOME\tts_voices\<音色名>\
├─ tts.json                    # ★必需：引擎、文件、参考音频、参数、来源/许可
├─ gpt\<音色名>-e15.ckpt        # GPT-SoVITS：GPT(T2S) 权重（v2Pro 微调产物）
├─ sovits\<音色名>_e8_s1336.pth # GPT-SoVITS：SoVITS 权重（v2Pro 微调产物）
├─ refs\                        # 参考音频 + 文本（每套 = 一个"语气"）
│  ├─ ref_zh_calm_01.wav
│  ├─ ref_zh_calm_01.txt        # 该音频的逐字文本（必需）
│  └─ ref_zh_angry_01.wav/.txt
├─ preview\demo_zh.wav          # 可选：预渲染试听（未加载模型也能听）
├─ training\                    # 可选：训练快照/指标（dataset.json、metrics.json、log.txt）
└─ license\VOICE_SOURCE.txt     # 可选但强烈建议：声音来源/授权声明
~~~

**`tts.json` 字段（schema `ams-tts-voice/1`）**

| 字段 | 类型 | 必填 | 说明 |
|---|---|---|---|
| `schema` | string | ✅ | 固定 `"ams-tts-voice/1"` |
| `name` / `id` | string | ✅ | 显示名 / 稳定 UUID（见 07 文档 §6.3 双 ID 策略） |
| `engine` | enum | ✅ | `gpt_sovits` / `qwen3tts` / `cosyvoice3` / `indextts` |
| `engine_version` | string | ✅ | 例 `20250606v2pro`；加载前校验兼容 |
| `language` | string[] | ✅ | 主语言（zh/en/ja/ko/yue/…） |
| `files` | object | ✅ | `{gpt, sovits}` 或 `{model_dir}`（Qwen 引用基础模型 + 可选 adapter） |
| `refs` | array | ✅ | 每项 `{audio, text, language, emotion_label, duration_sec, tags[]}`；**至少 1 条** |
| `defaults` | object | ✅ | `{ref_id, speed, pause, split_method, top_k, top_p, temperature, seed?}` |
| `link` | object | ⭕ | `{rvc_voice_id?, rvc_voice_name?, svs_voicebank_id?}` 同角色关联 |
| `source` | object | ⭕ | `{type: self/trained/imported/web, author?, url?, note?}` |
| `license` | object | ⭕ | `{voice_source_commercial: true/false/unknown, evidence?, model_license}` |
| `training` | object | ⭕ | `{pipeline, epochs_s1, epochs_s2, batch, dataset_sec, metrics{}}` |
| `hash` | object | ⭕ | `{gpt, sovits, refs[]}` sha256（导入时计算，用于重复检测） |
| `created_at/updated_at` | string | ✅ | ISO8601 |

**兼容策略**
- 从本机既有 GPT-SoVITS 整合包迁移（`GPT_weights_v2Pro\\*.ckpt` + `SoVITS_weights_v2Pro\\*.pth` + `standard\\` 或 `voice_configs\\*`）走「导入向导」：解析 `info`（epoch/step）、`config`（version/sr/n_speakers）、参考音频时长，生成 tts.json（P0 必备，用来接住用户已有资产）。
- 支持 UVP（`manifest.json` 含 `backends[].type=gpt_sovits/qwen3_tts`）作为**可选导入格式**：只取所需文件 + refs，转换成本地格式；不把 UVP 作为 TTS 运行时格式（00 第 6 节 未把 UVP 定为强制格式）。
- 不支持 v1/v2/v3/v4/v5 与 v2Pro **混放同一音色**；一个 tts 音色包绑定一个引擎版本，多版本用多个音色包（名称后缀提示）。

### 6.2 项目侧目录（配合 00 第 7 节）

~~~text
AMS_HOME\projects\<project-id>\
├─ tts\
│  ├─ segments.json        # 段落清单：角色/语气/文本/参数/产物/状态/耗时
│  ├─ <seq>_<voice>_<hash>.wav
│  └─ batch\               # 批量导入的脚本与 manifest
~~~

### 6.3 SQLite 表（Gateway 侧，按 00 第 8 节 任务持久化）

| 表 | 关键字段 | 说明 |
|---|---|---|
| `tts_jobs` | job_id, project_id, voice_id, engine, params_json, status, rt_factor, error_code | 与通用 jobs 表 1:1 扩展 |
| `tts_segments` | job_id, seq, text, voice_id, ref_id, out_path, dur_sec, cer, status | 长文本/批量逐段记录，支持"仅重生成失败段" |
| `tts_compare` | text, params_json, out_paths[] | A/B 快照 |
| `tts_text_cache` | text_hash, normalized, warnings_json | 文本校验缓存 |

### 6.4 音色与引擎的匹配规则

| 请求 | 匹配 | 结果 |
|---|---|---|
| voice.engine=gpt_sovits 且有 gpt/sovits 文件 | 直接用 v2Pro | ✅ |
| voice.engine=qwen3tts（仅 refs） | 用 Qwen3-TTS Base 零样本 | ✅ |
| 指定引擎但音色不兼容 | 禁止静默降级 | 报错 `VOICE_ENGINE_MISMATCH` + 建议 |
| 未指定引擎 | 按 voice.engine → 全局默认（gptsovits） | 记录到 job.engine（可追溯） |
| GPU 不可用 | v2Pro 走 CPU（RTF≈3.8）或 Qwen 拒绝（过大） | UI 明示「CPU 模式，较慢」【决策】CPU 默认允许但需确认 |

---


## 7. 接口与任务（Gateway）

### 7.1 同步/查询接口

| 方法 | 路径 | 作用 | 说明 |
|---|---|---|---|
| GET | `/api/v1/tts/engines` | 引擎状态 | 返回每个后端：是否安装、版本、许可类型、显存需求、健康、是否需许可确认 |
| GET | `/api/v1/tts/voices` | TTS 音色列表 | 透传音色库 API（`kind=tts`），含 refs/默认参数/引擎角标 |
| GET | `/api/v1/tts/presets` | 参数预设 | 内置（标准/稳重/活泼/慢速旁白…）+ 用户自定义（存 config） |
| POST | `/api/v1/tts/validate-text` | 文本预检（同步，≤100 ms） | 返回 `{normalized, segments_estimate, warnings[], suggestions[], risk(cn_罕见/多音字/英文/数字/超长)}` |
| GET | `/api/v1/tts/jobs/{id}/segments` | 分段明细 | 批量/长文逐段状态、时长、CER |
| POST | `/api/v1/tts/export` | 导出（同步/小任务） | 单段格式转换、合并、按角色分轨、生成 SRT（P1） |

### 7.2 Job 类型与参数

| Job 类型 | 用途 | 关键参数 | 产物 | 可恢复 |
|---|---|---|---|---|
| `tts.generate` | 单段/长文合成 | 见下表 A | `artifacts[]: wav + meta.json` | ✅（段级） |
| `tts.batch` | 批量脚本合成 | 见下表 B | `wavs + manifest.csv + report.json` | ✅（逐条） |
| `tts.train` | GPT-SoVITS 微调 | 见下表 C | `tts_voices/<名>/（ckpt+pth+refs+training/）` | ✅（按 epoch/step） |
| `tts.eval` | 生成质量评测（P1） | `segments/hash, metrics[]` | `metrics.json（CER/DNSMOS/UTMOS/相似度/RTF）` | ❌ |

**表 A：`tts.generate` 参数**

| 参数 | 类型 | 必填 | 默认 | 说明 |
|---|---|---|---|---|
| `text` / `text_file` | string / path | 二选一 | — | 单段文本 ≤1000 字；或 txt 文件（自动分段） |
| `voice_id` | uuid | ✅ | — | 来自音色库；不允许直接给路径（安全） |
| `ref_id` | string | ⭕ | voice.defaults.ref_id | 选择"语气"= 参考音频 |
| `engine` | enum | ⭕ | voice.engine | `auto` 时按 §6.4 匹配 |
| `language` | enum | ⭕ | voice.language[0] | v2Pro 建议显式 zh |
| `params` | object | ⭕ | voice.defaults | `speed(0.5–2.0)/pause(0–5)/split_method(cut0–5)/top_k(1–100)/top_p(0–1]/temperature(0–2]/seed(-1=随机)/repetition_penalty` |
| `mode` | enum | ⭕ | `normal` | `preview`（≤60 字快速试听）/`normal` |
| `output` | object | ⭕ | `{format:"wav", sr:32000, dir:"auto"}` | `format: wav/flac/mp3`；`sr: 32000/44100/48000` |
| `project_id` | string | ⭕ | — | 给了就落项目 tts/ 并写 project.json |
| `priority` | enum | ⭕ | `normal` | `high`（试听）/`normal`/`low`（批量） |

**表 B：`tts.batch` 参数**

| 参数 | 类型 | 说明 |
|---|---|---|
| `script` | object | `{format:"csv"/"jsonl"/"txt", content?（≤5 MB）/ path?}` |
| `script.columns` | string[] | CSV 列映射：`text,voice,ref,params,id`（灵活命名） |
| `defaults` | object | 缺省 voice/ref/params；行内可覆盖 |
| `stop_on_error` | bool | 默认 `false`：失败继续，逐条记录；`true`：首错即停 |
| `concurrency` | int | 固定 1（模型串行）；GPU 锁决定排队 |
| `output` | object | 同表 A；批量强制输出目录为项目/指定目录 |

脚本行示例（CSV，**非代码，仅格式示意**）：
`id,voice,ref,text,params_json` → `s001,玛拉妮,平静,"快把手搭在我手背上。","{\"speed\":1.0}"`。

**表 C：`tts.train` 参数**

| 参数 | 类型 | 默认 | 说明 |
|---|---|---|---|
| `voice_name` | string | — | 新音色名（落地目录名，校验 Windows 非法字符） |
| `engine` | enum | `gpt_sovits` | 一期只支持 v2Pro 微调 |
| `dataset` | object | — | `{audio_paths[] 或 dir, transcript_source:"asr"/"provided", text_file?}`；来自数据集插件 |
| `s1` | object | `{epochs:15, batch:8, precision:"16-mixed", max_sec:54, save_every:5}` | GPT 训练（沿用 v2Pro 默认量级） |
| `s2` | object | `{epochs:8, batch:8, lr:1e-4, fp16:true, save_every:4}` | SoVITS 训练 |
| `ref_pick` | object | `{mode:"auto", count:3}` | 训练后自动从数据集中挑 3–10 s 片段做参考音频（含 ASR 文本） |
| `resume_from` | object | null | `{s1_step?, s2_epoch?}` 断点续训 |
| `gpu` | string | `auto` | 由 GPU 锁与显存判定 |
| `eval_after` | bool | `true` | 训练后跑 `tts.eval`（ASR 回译 + DNSMOS） |

### 7.3 SSE 事件（`GET /api/v1/jobs/{id}/events`，遵循 00 第 8 节）

| 事件 | 负载 | 用途 |
|---|---|---|
| `progress` | `{percent, phase:"load"/"preprocess"/"synthesize"/"postprocess"/"train_s1"/"train_s2"/"eval", detail}` | 总进度 |
| `log` | `{level, message}` | 引擎原始日志（过滤绝对路径） |
| `segment.done` | `{seq, dur_sec, artifact_id, rt_factor}` | 逐段返回，先听先得 |
| `audio.delta` | `{seq, chunk_b64, final:false}` | 仅 Qwen3-TTS 流式（P1） |
| `artifact` | `{id, kind:"wav"/"flac"/"mp3"/"srt"/"json", size, sha256}` | 产物登记 |
| `status` | `{status, error?}` | 终态 |

### 7.4 错误码

| 错误码 | HTTP | 含义 | 用户可见提示/修复动作 |
|---|---|---|---|
| `TTS_ENGINE_MISSING` | 409 | 后端未安装 | 「先在启动器安装 GPT-SoVITS 服务」+ 跳转 |
| `TTS_MODEL_LOAD_FAIL` | 500 | 权重加载失败（版本不符/文件损坏） | 显示校验报告 + 「打开音色详情」 |
| `TTS_VOICE_NOT_FOUND` | 404 | 音色不存在/被删 | 刷新音色库 |
| `TTS_VOICE_ENGINE_MISMATCH` | 400 | 音色与引擎不兼容 | 建议切换到对应引擎 |
| `TTS_REF_AUDIO_INVALID` | 400 | 参考音频时长不在 3–10 s / 声道/格式问题 | 提供"裁剪到 6 s"一键建议 |
| `TTS_TEXT_EMPTY` / `TTS_TEXT_TOO_LONG` | 400 | 空文本 / 超 1000 字 | 自动分段按钮 |
| `TTS_TEXT_UNSUPPORTED` | 400 | 生僻字/特殊符号无法注音 | 标红该字符 + 建议替换 |
| `TTS_GPU_OOM` | 507 | 显存不足 | 提示降低 batch/切分/关闭其它服务 |
| `TTS_TRAIN_DATASET_INVALID` | 400 | 数据不足/文本缺失/采样率异常 | 打开数据集校验详情 |
| `TTS_LICENSE_NOT_ACCEPTED` | 403 | 黄色级后端未确认许可 | 弹许可确认页 |
| `TTS_CANCELLED` | 499 | 用户取消 | 保留已完成段 |
| `TTS_TIMEOUT` | 504 | 超过任务上限（默认 30 min/段集） | 建议分段/升温重试 |

---

## 8. AI 工具与 Skills

### 8.1 Host 工具（`music_tts_*`，每个都有 JSON Schema、超时、错误码）

| 工具 | 参数（要点） | 返回 | 超时 | 说明 |
|---|---|---|---|---|
| `music_tts_engines` | — | 引擎状态数组 | 5 s | AI 先查可用性再规划 |
| `music_tts_voices` | `{query?, engine?, language?}` | 音色摘要数组（id/名称/refs/引擎） | 10 s | 与音色库工具复用 |
| `music_tts_validate_text` | `{text, voice_id?}` | 警告/建议/预估分段 | 10 s | 生成前必跑，省一次失败 |
| `music_tts_split_script` | `{text, speakers[], max_chars, split:"line"/"sentence"/"paragraph"}` | 片段数组（角色/文本/建议 ref） | 30 s | AI 可在 Host 侧先拆后合成 |
| `music_tts_generate` | 表 A 核心字段（text/voice_id/ref_id/params/mode） | `{job_id}` | 30 s（提交） | 不阻塞等待；用 job 查询 |
| `music_tts_batch` | 表 B（script 内容/路径） | `{job_id, count_estimate}` | 30 s | 批量 |
| `music_tts_job` | `{job_id}` | 状态/进度/产物 | 10 s | 通用查询（也可用 music_job_*） |
| `music_tts_preview` | `{voice_id, ref_id?, text?}` | `{url, dur}` 或 job | 120 s | 快速试听（≤60 字） |
| `music_tts_compare` | `{text, variants[{name,params}]}` | 对比结果（多个 url + 指标） | 30 s（提交） | A/B |
| `music_tts_train` | 表 C 核心字段 | `{job_id}` | 30 s（提交） | 训练/微调 |
| `music_tts_train_status` | `{job_id}` | 阶段/epoch/损失/剩时 | 10 s | 训练监控 |
| `music_tts_export` | `{job_id/segments, format, sr, merge, srt}` | 产物路径 | 60 s | 导出 |

### 8.2 Skills（随插件分发 markdown）

| Skill | 触发场景 | 内容要点 |
|---|---|---|
| `ams-配音工作流` | "帮我把这段台词配成 XX 的声音" | 选音色 → 文本校验 → 分段 → 生成 → 试听 → 导出；参数默认表；失败处理 |
| `ams-配音参数调优` | "语气太平/太奇怪/太慢" | speed/pause/temperature/top_k/ref 选择的因果与建议（附本机实测 RTF） |
| `ams-配音批量剧本规范` | CSV/JSON 脚本格式、命名、错误行修复 | 模板与校验规则 |

**AI 操作规范（写入 skill）**：① 生成前必须调 `music_tts_validate_text`；② 不擅自使用黄色级后端；③ 不生成"模仿真实特定公众人物"的内容（许可红线，§10）；④ 批量必须先估算时长/显存并告知用户；⑤ 产物必须落工程或用户指定目录，不留临时文件。

---

## 9. 错误处理、降级与性能

### 9.1 降级链

| 故障 | 降级 | 用户提示 |
|---|---|---|
| v2Pro 服务崩溃 | 自动重启（≤2 次）+ 任务重排；仍失败 → 建议用 Qwen3-TTS 零样本（若有 refs） | 「主引擎异常，已切换快捷引擎（音色可能不同）」【决策】允许显式降级，默认不静默 |
| 显存不足 | 降 batch=1 → 关闭并行 → 提示关闭其它 GPU 任务 | 排队原因 |
| GPU 不可用 | CPU 模式（v2Pro 可跑，RTF≈3.8；Qwen3-TTS 1.7B 不建议） | 首次切换需确认（速度提示） |
| 模型缺失 | 阻止生成 + 引导安装（启动器组件中心） | 体积/许可展示 |
| 参考音频缺失 | 用 voice 其它 refs；都无 → 报错 | 建议补参考音频 |
| 文本过长 | 自动分段（cut3）+ 逐段生成 | 显示段数与预计时长 |

### 9.2 性能设计

| 项 | 措施 | 目标 |
|---|---|---|
| 冷启动 | 服务常驻 + 启动即加载 + 上次模型名缓存；加载进度 SSE | 冷启 ≤ 90 s（GPU） |
| 重复生成 | 文本+参数+音色 hash 缓存命中直接复用（可开关） | 命中 < 1 s |
| 长文本 | 分段并行度固定 1；先出首段 | 首段 ≤ 5 s（GPU，短句） |
| 批量 | 逐条进度；失败列表可重试；不整批重跑 | 100 条剧本 ≤ 10 min（GPU）【推断】 |
| 预览 | 预渲染 demo 存 cache；GPU 低峰预热 | 打开音色详情即可听 |
| 大库音色 | 音色列表分页 + 元数据缓存（音色库插件负责） | 1000 音色首屏 < 1 s |

### 9.3 资源与并发

- GPU 锁：所有 `tts.generate/batch/train/eval` 声明显存需求（v2Pro 推理 3–5 GB、训练 5–8 GB、Qwen3-TTS 4–8 GB），由 Gateway 统一排队（00 第 8 节）。
- 实时变声会话优先（00 第 8 节）；TTS 任务在会话期间只排队不抢占。
- 日志脱敏：不含用户绝对路径、不含文本全文（只留 hash 与前 20 字）。

---

## 10. 安全、许可与合规

### 10.1 许可矩阵（只列推荐组合；一期实际下发/内置范围）

| 级别 | 后端/组件 | 许可 | 一期处理 |
|---|---|---|---|
| **绿（可内置/可商用分发）** | GPT-SoVITS（MIT）、Qwen3-TTS（Apache-2.0）、CosyVoice3（Apache-2.0）、MOSS-TTS（Apache-2.0）、VibeVoice 权重（MIT，待核实 Quick Try 状态）、faster-whisper/WhisperX/FunASR（MIT）、librosa/pyloudnorm（ISC/MIT） | ✅ | 允许随安装器提供"组件下载"（仍不内置权重，用户确认下载） |
| **黄（需法务/显式确认）** | IndexTTS-2.5（bilibili 模型使用许可，<1 亿 MAU、禁止用于改进其他模型）、MegaTTS3（权重获取条件待核实） | ⚠️ | **默认不下发**；用户自行安装 + 首次使用弹许可原文 + 记录"已确认"；UI 标注"实验室" |
| **红（禁止内置/分发）** | F5-TTS 权重（CC-BY-NC-4.0）、Spark-TTS 权重（CC-BY-NC-SA-4.0）、Fish-Speech（社区/研究许可）、Higgs TTS（研究）、Breeze-TTS-2（研究）、ChatTTS（AGPL-3.0）、so-vits-svc/diff-svc（AGPL/GPL） | ❌ | 不进组件清单；不写进"推荐组合"；如用户强需求只做"外部程序"说明，不集成 |
| 红线（音色/数据） | 任何**未授权**的真实人物声音、受版权保护的配音/歌曲素材 | — | 导入/训练前必须勾选"我有权使用"；记录到 tts.json/license；导出加 AI 标识 |

### 10.2 合规动作（对应 00 第 12 节）

| 动作 | 实现 |
|---|---|
| AI 生成标识 | 导出默认写元数据（`ams.ai_generated=true, engine, voice, date`），可选文件名后缀与 ID3 注释（mp3）；UI 明确提示 |
| 声音来源授权 | 训练/导入向导强制填写 `source.type`（self/授权/未知）；`unknown` 时训练仍可做但导出提示风险 |
| 参考音频授权 | refs 记录来源；系统内置示例音频（用于音色库试听）必须为可商用授权素材（自录/CC0） |
| 不提供"克隆真人"引导 | skill 与 UI 中不做"克隆某明星/主播"模板；提示不得用于冒充、诈骗、侵权 |
| 日志隐私 | 不记录完整文本与音频；仅 hash + 长度 |
| 法规提示 | 首次训练/导出展示《互联网信息服务深度合成管理规定》《生成式人工智能服务管理暂行办法》《人工智能生成合成内容标识办法》要点与用户责任（条文以官方文本为准） |

---

## 11. 测试与验收

### 11.1 测试矩阵

| 类型 | 用例 | 通过标准 |
|---|---|---|
| 单元 | tts.json schema 校验、文本切分（cut0–5）、文本预检规则、参数范围钳制、错误码映射 | 覆盖率 ≥ 70%；边界值全过 |
| 契约 | Gateway ↔ 服务：任务参数/SSE 事件/产物登记/取消（杀进程树） | 端到端 20 例 |
| 集成 | v2Pro 单段/长文/批量；Qwen3-TTS 零样本/流式；训练 1 分钟数据全流程 | 全部场景 S1–S6 通过 |
| 质量回归 | 固定 20 句中文 + 固定 seed 的「金标准集」；每次升级跑 CER/DNSMOS/相似度/人工抽听 | CER ≤ 5%（GPU）、DNSMOS OVRL ≥ 3.0、人工 MOS ≥ 3.5（5 分制） |
| 兼容 | v2Pro 官方底模 + 本机 Mualani 微调模型 + 训练新模型；缺 refs/缺文件/坏文件 | 全部给出可读错误而非崩溃 |
| 性能 | CPU/GPU RTF、冷启动、100 条批量、长文 3000 字 | 达 §9.2 目标 |
| 安全 | 恶意 tts.json（路径穿越/超大文件/非法字符）、未确认许可后端、无授权声明 | 全部拦截 |
| 可用性 | 空态、许可弹窗、失败恢复、任务取消后清理 | 用户可独立完成 S1–S6 |

### 11.2 验收清单（发布门禁）

1. 三个绿灯后端可用且版本锁定；红黄名单在代码/配置中硬编码拦截。
2. 训练向导能从数据集到「可试听音色」全流程跑通（含断点续训）。
3. 批量脚本支持 ≥100 条，失败可重试，产物 + manifest 完整。
4. AI 工具全部可从对话调用并返回可读结果；skill 生效。
5. 导出带 AI 标识；许可提示可见；日志无隐私泄漏。

---

## 12. 里程碑与工作量估算

| 里程碑 | 内容 | 交付物 | 估算（人日） | 依赖 |
|---|---|---|---|---|
| M0 | 契约冻结 | Job 参数/SSE/错误码/tts.json schema + mock 服务 | 3 | 00 定稿 |
| M1 | v2Pro 推理服务 | `ams-svc-gptsovits`：单段/长文/批量、GPU/CPU、预热、健康检查 | 10 | 组件安装（启动器） |
| M2 | 工作台 UI | 编辑器/角色选择/语气/参数/试听/任务 | 8 | core 音色选择器 |
| M3 | 批量 + 导出 + 项目集成 | 脚本、manifest、合并/分轨、SRT（P1）、project.json | 7 | project 插件 |
| M4 | 训练向导 | 数据集接入、ASR 打标校对、s1/s2、断点续训、自动测评与入库 | 10 | 数据集插件、trainer 经验 |
| M5 | Qwen3-TTS（P1） | 零样本 + 流式 + 许可确认 | 6 | 组件分发 |
| M6 | 评测/许可/加固 | tts.eval、许可拦截、回归集、压测 | 5 | 评测器插件（P1） |
| **合计** | | | **≈ 49 人日（≈ 10 周单人 / 5 周双人）** | |

> 说明：不含启动器组件分发、GPU 驱动/环境与 Qwen3-TTS 权重下载链路（属 01 启动器与 02 主体）；不含 IndexTTS/CosyVoice3（P2）。

---

## 13. 风险与开放问题

### 13.1 风险【风险】

【风险】下表按「影响 × 概率」排序，R1–R9 均需在对应里程碑内闭环（见 §12）。

| # | 风险 | 影响 | 概率 | 对策 |
|---|---|---|---|---|
| R1 | GPT-SoVITS 引擎强依赖工作目录/单例/无并发，服务化封装不彻底 | 崩溃、并发错乱 | 中 | 独立进程 + 固定 cwd + 串行队列 + 每次任务子进程隔离（超时可杀）；不共享模型给其它插件 |
| R2 | v2Pro 版本漂移（V5 分支、上游 2026 仍在提交） | 兼容破坏 | 中 | 锁 tag + 服务镜像锁依赖 + 保留老版本后端回退 + 「金标准集」回归 |
| R3 | 训练数据版权/声音权 | 法律风险 | 高 | 授权声明 + 来源记录 + AI 标识 + 不做真人克隆模板 |
| R4 | 显存竞争（变声实时优先） | 排队、体验差 | 中 | GPU 锁 + 预估时间 + 低峰预热 + CPU 降级 |
| R5 | 参考音频 3–10 s 限制导致用户频繁踩坑 | 生成失败 | 高 | 导入/选择前置校验 + 一键裁剪 + UI 示例 |
| R6 | 文本前端（多音字/生僻字/中英混排）效果不稳 | 读错字 | 中 | 文本预检 + 用户可编辑注音（P1）+ CER 自动复检 |
| R7 | Qwen3-TTS 为 2026 新模型，API/权重格式变动快 | 集成返工 | 中 | 版本锁 + 适配层隔离 + 只作为"快捷后端" |
| R8 | 许可误用（把黄色/红色后端打进安装包） | 法务 | 低 | 代码硬编码 allowlist + CI 检查依赖清单 + 发布前许可证扫描（§14 集成） |
| R9 | CPU 降级体验差（RTF≈3.8） | 用户流失 | 中 | 明确提示 + 仅小段/试听 + 建议 GPU |

### 13.2 开放问题（【待定】）

| # | 问题 | 选项 | 建议 |
|---|---|---|---|
| Q1 | 一期是否内置 Qwen3-TTS 0.6B（约 1.2 GB）作为默认快捷后端？ | 0.6B 轻量 / 1.7B 质量 / 都不内置 | 建议内置 0.6B（体积可接受、CPU 亦能跑），1.7B 按需下载 |
| Q2 | 默认引擎 | v2Pro 优先 / Qwen 优先 | v2Pro（有微调即用），无微调音色时自动 Qwen 零样本 |
| Q3 | tts_voices 是否必须 tts.json？ | 强制 / 兼容无清单目录 | 强制（TTS 格式非 RVC 标准，缺清单无法可靠加载） |
| Q4 | 时间线（P1）做到什么程度 | 仅片段排序/间隔 / 对齐视频口型 / 多轨 | 一期只做"片段序列 + 时间码 + SRT" |
| Q5 | 是否支持 `aux_ref_audio_paths`（多参考融合） | 支持 / 不支持 | P1 支持（v2Pro 原生能力，可做"混合音色"） |
| Q6 | 训练数据下限 | 1 min / 3 min / 5 min | 1 min 允许（带质量警告），3 min 为推荐值 |
| Q7 | 是否接入 IndexTTS-2.5 | 是 / 否 | 否（一期），等法务确认与用户需求 |
| Q8 | 是否提供 TTS 训练的全参数面板（高级） | 简单 3 档 / 全参数 | 默认 3 档（快速/标准/高质量），高级折叠 |

---

## 14. 与启动器 / 内核 / 其他插件的集成点

| 对象 | 集成内容 | 契约 |
|---|---|---|
| 00 总体 | 目录（`tts_voices/`）、任务规范、GPU 锁、许可分级 | 遵循；本分册新增项已列 §5.4 |
| 01 启动器 | 组件安装（`service.gptsovits`/`service.qwen3tts`）、模型下载（底模/参考示例）、诊断（显存/驱动）、许可展示 | 组件 ID/SemVer 见 §5.2；下载校验 sha256（00 第 12 节） |
| 02 主体（内核） | Host/Client 插件装载、页面槽位、工具注册、SSE 订阅 | 00 第 9 节 槽位表 |
| 03 变声器 / 04 歌曲换源 | 音色关联：同一角色的 RVC 音源 ↔ TTS 音色（tts.json.link） | 音色库提供关联 API（07 §7） |
| 06 数据集工作室 | 训练数据（切片/ASR/校对/质量报告） | 数据集 job 产物目录 + `dataset.json` |
| 08 混音母带 | TTS 片段导入混音台（对齐/响度） | 产物 wav + segments.json 时间码 |
| 09 评测器 | CER/DNSMOS/相似度/RTF 指标 | `tts.eval` 产物 metrics.json |
| 10 项目管理 | project.json 记录 tts 片段、参数、模型版本 | 00 第 7 节 |
| 07 音色库 | 音色 CRUD/选择器/预览/导入 | 07 文档 §7 的 `/voices/*` |
| Gateway | 任务类型注册、GPU 锁、产物库、日志 | 00 第 8 节；新增 `tts.batch`/`tts.eval` 需登记 |

---

## 附录 A：一页速查

| 项 | 值 |
|---|---|
| 插件 ID | `ams.music.tts`（Host+UI，页面「配音 TTS」） |
| 后端 | `gptsovits`（v2Pro，P0）/ `qwen3tts`（P1）/ `cosyvoice3`、`indextts`（P2） |
| 目录 | `AMS_HOME/tts_voices/<名称>/（tts.json + gpt/ + sovits/ + refs/ + preview/ + training/）` |
| 任务 | `tts.generate` / `tts.batch` / `tts.train` / `tts.eval` |
| 工具 | `music_tts_engines/voices/validate_text/split_script/generate/batch/job/preview/compare/train/train_status/export` |
| 关键约束 | 参考音频 3–10 s；文本 ≤1000 字/段；输出 32 kHz；GPU 锁串行 |
| 许可红线 | 绿：MIT/Apache（GPT-SoVITS/Qwen3-TTS/CosyVoice3）；黄：IndexTTS-2.5；红：F5/Spark/Fish/Higgs/ChatTTS |

## 附录 B：参考与证据

- 本机实测：`02_ai配音_GPT-SoVITS/报告.md`（v2Pro 环境、3–10 s 限制、32 kHz 输出、RTF≈3.8、SoVITS 文件头 b"05"）
- 选型/许可/版本：`05_新方案探讨/03_开源栈调研/报告.md` §5、§6.3、§10、§11
- UVP 与声库格式：`03_开源栈调研/报告.md` §8；`可行性探讨.md` §6.2
- 上游：GPT-SoVITS https://github.com/RVC-Boss/GPT-SoVITS （tag 20250606v2pro）；Qwen3-TTS https://huggingface.co/Qwen/Qwen3-TTS-12Hz-1.7B-Base
- 合规：《互联网信息服务深度合成管理规定》（2023-01-10）、《生成式人工智能服务管理暂行办法》（2023-08-15）、《人工智能生成合成内容标识办法》（2025-09-01）（条文以官方文本为准）
