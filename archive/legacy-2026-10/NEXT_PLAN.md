# AMS Studio 下一阶段改造计划

> **注意：本文 v1 对“歌曲换源 / 变声器 / MSVC”的职责理解有偏差，已被 `AMS-Studio-B方案完整设计与实施计划-v2.md` 取代；请以 v2 为准。**

> 版本：v0.1（2026-10-24）  
> 目标：按用户 2026-10-24 的需求，修正现有功能、补齐数据目录、重做“歌曲换源”与 MSVC 边界、增加启动器介绍页、补齐参数与未完成功能。  
> 原则：先修正确性，再做体验；数据目录只创建、不删除；所有目录和文件路径由程序从安装根 data 推导，UI 不写死绝对路径；每个阶段可独立验收、可回滚。

---

## 0. 结论先行

这次需求不是一个小补丁，而是四类工作叠加：

1. **现有功能修复**：RVC 变声当前是坏的；MSVC 被错误地当成“歌曲换源”。
2. **数据与目录体验**：TTS 音源独立目录、打开文件夹按钮、data 目录启动时自动补齐。
3. **内容与介绍**：启动器顶部完整介绍页，包含所有功能、上游来源、GitHub 和版本。
4. **未完成能力**：音源炼制、歌曲换源、参数面板，以及设计文档中尚未实现的插件。

建议按 6 个阶段交付，不要一次性改完再验收。阶段 1、2 可以最快让当前安装变得可用；阶段 3–6 是功能扩展，需要分别验收。

---

## 1. 已确认的问题与证据

### 1.1 RVC 变声无法使用

用户操作记录中的失败任务：

~~~text
job_3ee4708aba3b41bc  type=convert.rvc
RunnerError: 缺少参数 voice_name（音源名或 pth 路径）
~~~

当前 plugin-ui 的“变声器”表单：

- “目标音色”下拉框的值来自 /ams/api/voices。
- 表单初始值是空字符串。
- 下拉框没有“请选择”占位项；浏览器视觉上会默认显示第一项，但 React 状态仍为空。
- 提交时 onSubmit 会跳过空字符串，因此 Gateway 收到的参数里没有 voice_name。

结论：不是 RVC 后端坏了，而是 UI 的默认值/必填校验有缺陷。同类问题也存在于：

- “人声分离”的 model 下拉框：初始为空，提交可能缺少 model。
- “歌曲换源”的 out_dir：UI 提示可留空，但 Gateway 强制要求，提交会报缺少 out_dir。

修复方案：

- 所有静态/动态下拉框增加明确的空占位项。
- 动态音源列表加载后，如果当前值为空，自动选中第一项（或要求用户明确选择）。
- 提交前做必填校验，直接给出中文提示，不再把空参数发给 Gateway。
- 后端错误保留，作为最后一道防线。

### 1.2 “歌曲换源”把 MSVC 转换错当成换声

当前代码事实：

- 插件页“歌曲换源”页签的 jobType 是 convert.msvc。
- 页面说明写“用已炼制模型替换歌曲音色”。
- Gateway 的 convert.msvc 实际执行 msvc2pth.py：输入 .msvc 容器，输出 .pth + conversion.json。
- 它不接收歌曲音频，也不输出换声后的歌曲。
- 设计文档中的“歌曲换源”明确是 RVC 离线换声：分离 → 干声 RVC → 与伴奏混合；任务类型应是 convert.rvc，不是 convert.msvc。

结论：“歌曲换源”页签接错了后端；MSVC 只是模型格式转换工具，不是音乐生产主流程。

MSVC 工具还有一个边界：

- ams-tools/msvc2pth/README.md 明确写“不生成 .index”。
- MSVC 容器只包含模型权重，不包含检索 index。
- 因此“把 MSVC 转成 pth 和 index”在现有格式下只能做到：
  - 生成 pth；
  - 如果用户另外提供同名 index，一并放入音源目录；
  - 没有 index 时使用 index_rate=0，而不是伪造一个 index。

这一点需要在 UI 和介绍页写清楚，不能继续承诺“自动生成 index”。

### 1.3 当前真实功能边界

已经实现且已有真实 Runner：

- TTS 语音合成
- RVC 变声（后端可用，UI 默认值有 bug）
- pymss 人声分离
- MSVC → pth 转换
- 任务队列 / SSE / 取消 / 产物
- 外部素材导入到 data
- 音源扫描 / 默认曼波

未实现或未正确接线：

- 音源炼制（train.rvc）：页签是占位，Gateway 未注册。
- 歌曲换源（RVC 整曲）：被错误映射到 MSVC。
- 项目/数据集/混音/评测：设计文档有方案，当前没有页面和 Runner。
- 实时变声：设计文档有方案，当前没有服务。
- 模型市场/自动下载、产物下载按钮、自动更新：未实现。

---

## 2. 目标目录与数据契约

### 2.1 启动时自动创建的数据目录

启动器初始化时统一校验并创建；缺失则创建，已存在则保留，绝不删除用户文件：

~~~text
data/
  artifacts/                 任务产物
  assets/                    组件资产
  backups/                   备份/回收站
  cache/                     缓存
  components/                组件程序
  config/                     配置
  datasets/                  训练数据集
  kernel/                    内核
  logs/                      日志
  models/                    模型缓存（pymss 等的基础模型）
  projects/
    imports/                 外部素材导入
  runtime/                   后端运行时
  tmp/                       临时目录
  tools/                     工具
  tts_voices/                TTS 音源（新增，独立于变声音源）
  voices/                    变声音源（RVC）
~~~

新增的 TTS 音源目录不得放在 runtime/tts 版本目录里，避免组件更新覆盖用户文件。

### 2.2 TTS 音源目录

建议结构：

~~~text
data/tts_voices/
  zh_CN-huayan-medium/
    zh_CN-huayan-medium.onnx
    zh_CN-huayan-medium.onnx.json
  我的TTS音色/
    我的TTS音色.onnx
    我的TTS音色.onnx.json
~~~

规则：

- TTS 页面只展示 .onnx 模型；同名 .json 是必需配置文件。
- 首次初始化时，把 runtime 内置的 zh_CN-huayan-medium 复制到 data/tts_voices，作为可管理副本。
- Gateway /tts/models 合并扫描：
  - data/tts_voices
  - runtime/tts/0.1.0/models（内置兜底）
- 用户删除/替换 TTS 音源不影响 runtime。
- RVC 的 data/voices 不变，二者绝不混用。

### 2.3 打开文件夹按钮

UI 不显示绝对路径，只传逻辑类型：

- rvc_voices → data/voices
- tts_voices → data/tts_voices
- artifacts → data/artifacts

实现：

- 启动器：Tauri command open_data_folder(kind)，服务端解析路径并调用资源管理器。
- 插件页面：Host 路由 /ams/api/open-folder，只接受白名单 kind，解析后调用 explorer。
- 所有路径由 AMS_HOME 推导，UI 文案只写“变声音源 / TTS 音源 / 任务产物”。

按钮位置：

- 变声器页：打开变声音源目录
- TTS 页：打开 TTS 音源目录
- 任务（Gateway Job）区域底部：打开产物目录

---

## 3. 启动器介绍页

### 3.1 入口

启动器顶部增加一个明显的入口按钮，例如“使用说明 / 关于 AMS”。点击后在启动器内部打开一个大子页面，不是外部浏览器，也不是弹窗小窗。

### 3.2 子页面结构

左侧目录 + 右侧内容，支持返回启动器：

1. AMS Studio 简介与整体架构
2. 安装、初始化与目录
3. 启动器逐卡片说明
4. 内核（AMS Kernel / DSH）说明
5. 插件总览与启用方式
6. TTS 语音合成
7. RVC 变声器
8. pymss 人声分离
9. 歌曲换源（RVC 整曲换声）
10. MSVC → pth 工具
11. 音源目录与素材导入
12. 任务、进度、产物与错误
13. 参数手册（每个功能的参数、默认值、适用场景）
14. 上游项目、GitHub 地址、版本与许可证
15. 已知限制与常见问题

内容要求：

- 不要只写营销文案；每个按钮、字段、文件位置都要有说明。
- 参数部分必须来自对应上游项目当前版本的真实参数，而不是凭印象编写。
- 上游来源要列出：项目名、GitHub URL、使用版本/commit、许可证、AMS 改了哪里。
- 介绍页内容尽量做成数据文件或结构化常量，避免散落在 Vue 模板里。

### 3.3 上游来源清单（待核对）

| 组件 | 当前使用 | 上游项目 | 需要核对 |
|---|---|---|---|
| 内核 | DSH 0.2.1-rc.2 | https://github.com/deepseek-ai/deepseek-harness | 具体 fork commit、许可证 |
| 插件框架 | Cordis | https://github.com/cordiverse/cordis | 版本、许可证 |
| RVC 运行时 | RVC20260311 整合包，约基于官方 2.2.231006 | https://github.com/RVC-Project/Retrieval-based-Voice-Conversion-WebUI | 实际代码基线、MIT、二次修改 |
| pymss | pymss 2.1.9 / pymss-core 0.1.11 | https://github.com/pymss-project/pymss | 版本、MIT、模型权重许可 |
| Piper TTS | 当前 piper Windows 包 | https://github.com/rhasspy/piper（或当前官方 fork） | 二进制版本、模型许可 |
| MSVC 转换 | ams-tools/msvc2pth 0.4.0 | 无公开上游；逆向心月 MSVC 格式 | 不声称开源上游，注明自研 |
| 默认曼波音源 | 曼波 voicepack | 第三方音色资产 | 来源、授权、是否允许随包分发 |
| TTS 默认音色 | zh_CN-huayan-medium | Piper 模型来源 | 模型许可证 |

注意：不要为了“好看”把 RVC 版本写成官方最新版。当前 runtime 是 RVC20260311 基线，介绍页应如实写“当前运行时基于 2.2.231006 附近代码”，并把 2.3.260718 作为后续升级目标。

---

## 4. 分阶段实施计划

### 阶段 0：修复准备（已完成诊断）

- 确认 RVC 失败原因：UI 未提交 voice_name。
- 确认 MSVC/歌曲换源错位：页面映射到了 convert.msvc。
- 确认 MSVC 不生成 index。
- 确认当前 data 缺少 tts_voices、datasets、cache、backups 等目录；projects/imports 已存在。
- 确认启动器当前没有介绍页入口和打开文件夹按钮。

交付：本计划文件 + 诊断结论。

### 阶段 1：正确性与即时可用（建议 1–2 天）

目标：先让现有页签真正可用。

1. 修复所有下拉框：
   - 增加空占位项；
   - 动态音源加载后自动选择第一项；
   - 提交前校验必填项；
   - RVC 目标音色、pymss model 不再静默缺失。
2. 修复歌曲换源 out_dir 的提示与校验：
   - 要么改为必填；
   - 要么在 UI 明确生成 data 内默认目录并显示。
3. 启动器启动时调用 ensure_data_layout，补齐 data 目录。
4. 增加打开文件夹能力：
   - 启动器命令；
   - 插件 Host /ams/api/open-folder；
   - 变声音源、TTS 音源、任务产物三个按钮。
5. 新增 TTS 音源目录与 Gateway 扫描逻辑：
   - data/tts_voices；
   - 内置模型复制；
   - TTS 下拉显示两个来源。
6. 修“歌曲换源”页签的标签/说明，暂时不承诺 MSVC 换声。
7. 对 RVC、TTS、pymss 做一次真实提交回归，使用当前安装验证。

验收：

- 不手动填写目标音色也能选到曼波；
- RVC 任务成功生成 wav；
- pymss 任务成功生成 stems；
- 三个文件夹按钮都能打开正确目录；
- 启动器首次启动后 data 目录完整；
- UI 不再显示“歌曲换源使用 MSVC”的错误说明。

### 阶段 2：介绍页与内容体系（建议 2–3 天）

目标：完成启动器顶部大子页面，内容可维护。

1. 增加顶部入口与内部路由/视图切换。
2. 建立内容目录与组件：
   - 简介、安装、启动器、内核、插件、参数、上游、FAQ。
3. 完成上游版本/许可证核对表。
4. 将当前所有功能用法写入介绍页：
   - 每个按钮、字段、默认值、输出位置、常见错误。
5. 支持内部搜索、目录跳转、返回启动器。
6. 页面不写绝对路径；显示 data 相对结构。

验收：

- 启动器顶部可进入介绍页；
- 15 个章节都有内容；
- 每个已实现功能都能按页面独立操作；
- 上游 GitHub/版本/许可证可点击核对；
- 不出现“已实现”但实际是占位功能的错误描述。

### 阶段 3：MSVC 与歌曲换源重构（建议 3–5 天）

目标：彻底分开“模型格式转换”和“音乐换声”。

1. MSVC 工具独立：
   - 只有输入 .msvc、输出目录、可选 reference pth、名称；
   - 输出 pth + conversion.json；
   - 不生成 index；
   - 可选：允许用户手动关联同名 index，并在界面明确提示；
   - 不再放在“歌曲换源”主流程里。
2. 歌曲换源改为 RVC 整曲流程：
   - 输入整曲或干声；
   - 可选 pymss 分离；
   - RVC convert.rvc；
   - 可选与伴奏混合；
   - 输出到 data/artifacts；
   - UI 显示步骤与产物。
3. 长音频处理：
   - 先实现整段 RVC（当前 runtime 可跑）；
   - 后续按设计文档做切块、重叠、交叉淡化，避免长音频内存峰值。
4. 参数先复用现有 RVC 参数，后续阶段扩展。
5. 同步更新介绍页和说明书，纠正 MSVC 概念。

验收：

- MSVC 只能从独立工具进入；
- 歌曲换源不再要求 .msvc；
- 整曲换声能生成 wav；
- 分离 + 换声 + 混合流程可手工串通；
- 介绍页/说明书中不再把 MSVC 当换声后端。

### 阶段 4：参数面板与上游参数对齐（建议 2–4 天）

目标：把“能跑”升级到“可调”。

1. 从上游代码/文档核对参数：
   - RVC 当前 runtime 实际支持：
     - f0_up_key
     - f0_method：pm / harvest / crepe / rmvpe（以当前代码为准）
     - index_rate
     - filter_radius
     - resample_sr
     - rms_mix_rate
     - protect
     - index_path
   - pymss：model、device、chunk、overlap、ensemble、输出格式等（以 pymss 2.1.9 文档/CLI 为准）。
   - Piper：model、config、speaker、length_scale、noise_scale、noise_w、sentence_silence 等。
   - MSVC：input、out、name、reference-pth、no-cache（以 msvc2pth 0.4.0 为准）。
2. 后端 Runner 扩展参数校验与命令构建。
3. UI 增加“高级参数”折叠区，默认只显示常用参数。
4. 增加参数说明、默认值、范围、适用场景。
5. 增加参数模板/最近使用（先做 JSON 本地保存，后续再上 DB）。
6. 用短音频/长音频分别做回归。

验收：

- 每个参数有范围校验，非法值返回中文错误；
- 参数能在产物 sidecar/任务日志中追溯；
- 参数说明与上游版本一致；
- 不把当前 runtime 不支持的参数做成看起来可用。

---

### 阶段 5：音源炼制（train.rvc）（建议 3–7 天，取决于样本和 GPU）

目标：当前占位页变成真正的训练流程。

当前有利条件：runtime 中已有 RVC 完整训练代码和预训练权重（assets/pretrained_v2、hubert、rmvpe）。困难：训练是长任务，需要数据集预处理、特征提取、训练、index 构建、失败恢复和 GPU 排队。

计划：

1. 定义数据集目录和音频规范：
   - data/datasets/<名称>/
   - 至少 10 分钟低底噪语音；wav 优先；
   - 校验时长、采样率、声道、损坏文件。
2. 新增 Gateway train.rvc Runner：
   - 数据集扫描；
   - 特征提取；
   - 训练；
   - 保存权重；
   - 构建 index；
   - 注册到 data/voices/<名称>/。
3. 任务阶段：prepare → extract → train → index → register。
4. UI：训练数据集目录、模型名、采样率、版本、f0 方法、epochs、batch size、保存间隔、GPU。
5. 失败恢复：可取消、可重试、日志可查看。
6. 首批验收使用 10–20 分钟小样本，不直接跑长数据集。

验收：

- 从 data/datasets 选一个数据集可训练出 pth + index；
- 结果出现在 data/voices，并能被 RVC 使用；
- 取消任务能停止训练进程；
- 失败有明确日志；
- 不破坏已有音源。

风险：

- 训练耗时可能以小时计；
- GPU 显存/数据质量决定结果；
- 必须先验证 runtime 的训练入口与依赖完整，避免 UI 做了但后端跑不起来。

### 阶段 6：其余设计插件（建议按插件逐个交付，不建议一次性铺开）

设计文档中尚未实现的插件：

1. 数据集工作室
2. 音色库高级管理
3. 混音母带
4. 评测器
5. 项目管理
6. 实时变声

建议顺序：

1. 项目管理（把所有产物串起来，收益最大）
2. 数据集工作室（训练前处理，和阶段 5 配套）
3. 混音母带（歌曲换源后的成品导出）
4. 评测器（自动指标 + A/B）
5. 音色库高级管理（试听、标签、导入导出）
6. 实时变声（独立音频服务，难度最高，单独立项）

每个插件都必须有：

- 明确的输入/输出；
- Gateway Job 类型；
- 数据目录；
- UI 页面；
- 任务进度/取消/错误；
- 介绍页章节；
- 真实回归，而不是占位页。

### 阶段 7：整合、发布与验收

1. 更新 feed sequence、安装包、插件包。
2. 更新完整说明书与介绍页。
3. 干净 VM 从零安装验证。
4. 当前安装升级验证。
5. 数据目录初始化验证。
6. 回归 RVC / pymss / TTS / 歌曲换源 / MSVC 工具。
7. 检查 C 盘无 AMS 数据、无绝对路径写死、无环境变量依赖。

---

## 5. 关键设计决定（需要你确认）

### 决策 1：歌曲换源的定义

建议按设计文档定义为：

> 输入整曲或人声，先可选分离，再用 RVC 换声，最后可与伴奏混合导出。

不再使用 .msvc 文件。

请确认这是你要的“歌曲换源”。

### 决策 2：MSVC 的 index

现有 MSVC 格式不包含检索 index，工具也不会生成 index。建议：

- pth 正常生成；
- 若用户有同名 index，可手动放入 data/voices/<名称>/；
- 没有 index 时 index_rate=0；
- 不伪造 index。

请确认接受这个边界。

### 决策 3：未完成功能的范围

“那几个功能没做完的接着做”有两种理解：

A. 只完成当前五个页签中未完成/接错的部分（音源炼制、歌曲换源、参数）；
B. 把设计文档里的数据集、项目、混音、评测、实时变声也全部做完。

B 是多个独立项目，建议按阶段 6 逐个交付。请确认先做 A，还是 A+B 全排。

### 决策 4：训练和运行时版本

当前 runtime 是 RVC20260311（约 2.2.231006 基线），训练代码与预训练权重齐全。可以先在现有 runtime 上做 train.rvc，不必先升级到 2.3.260718。升级运行时风险高（依赖、显存、模型兼容），建议训练先基于现有 runtime。

---

## 6. 文件与代码触点

预计涉及：

- 启动器 ams-launcher/src/App.vue
- 启动器 src-tauri/src/lib.rs
- 插件 ams-plugins/packages/plugin-ui/src/client.js
- 插件 ams-plugins/packages/plugin-core/src/index.js
- Gateway app/api/tts.py
- Gateway app/runners/tts.py
- Gateway app/runners/rvc.py
- Gateway app/runners/pymss.py
- Gateway app/runners/msvc.py
- Gateway app/paths.py
- 新增 app/runners/train.py 与训练脚本
- 发布工具/feed/setup
- 说明书与介绍页内容

所有数据路径从 AMS_HOME/安装根 data 推导；UI 不出现绝对盘符。

---

## 7. 验收清单

### 数据与 UI

- [ ] 启动器初始化后 data 下所有目录存在
- [ ] TTS 音源目录独立且可打开
- [ ] RVC 音源目录可打开
- [ ] 任务产物目录可打开
- [ ] UI 无绝对盘符路径
- [ ] 介绍页可从顶部进入并返回

### 功能

- [ ] RVC 目标音色默认/必填正确，任务成功
- [ ] pymss model 默认/必填正确，任务成功
- [ ] TTS 默认模型和用户模型均能合成
- [ ] MSVC 工具只做格式转换，不再出现在歌曲换源
- [ ] 歌曲换源使用 RVC，不再要求 .msvc
- [ ] 参数面板与上游参数一致
- [ ] 音源炼制能产出 pth+index（若要完成此功能）

### 发布

- [ ] 新 feed 签名验证通过
- [ ] 新 setup 安装通过
- [ ] 干净 VM 全链路通过
- [ ] 旧安装升级通过
- [ ] 数据目录与用户音源升级不丢失

---

## 8. 建议的下一步

1. 先确认第 5 节的 4 个决策。
2. 我按阶段 1 做 correctness hotfix：RVC 默认值、pymss 默认值、data 初始化、打开文件夹、TTS 音源目录、歌曲换源标签纠正。
3. 阶段 1 通过后，再做介绍页和 MSVC/歌曲换源重构。
4. 不先动训练和大型插件，避免在基础流程未稳定时扩大风险。

---

## 9. 当前状态备注

- 本计划只描述改造，不修改现有 data。
- 当前安装已有上一轮认证/性能/进度条修复。
- 当前 RVC 失败的日志证据保留在 data/logs/gateway/gateway.log。
- 建议在阶段 1 开始前备份 data/voices、data/tts_voices（若已创建）、data/projects、data/artifacts。
