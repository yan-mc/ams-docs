# H2 核心功能回归结果（2026-10-25）

状态：**H2 核心功能回归完成。闲云标准音源已打包并通过最终中文 RVC/评测。**

## 1. 环境

- Gateway：127.0.0.1:10507（测试期间动态端口）
- Host：127.0.0.1:10567
- Kernel：运行中
- 生产 data：F:\AMS-Studio\data
- 测试夹具：F:\AMS-Studio\data\projects\imports

## 2. 逐模块结果

| 模块 | 测试 | 结果 |
|---|---|---|
| Kernel/DSH | Host summary、Kernel 崩溃恢复 | 通过 |
| Host | projects 创建、steps、export/import/trash | 通过 |
| Home | 12 个页签静态入口 | 通过 |
| Gateway | ping、health、取消、GPU 队列 | 通过 |
| TTS | 中英双模型、批量、缺模型、20001 字 | 通过（错误码正确） |
| RVC | 3/5/12 分钟、无 index、训练音色 | 通过 |
| pymss | 3/10 分钟、四 stems | 通过 |
| 训练 | 10/50 epoch、pth/index、注册、RVC | 通过 |
| 数据集 | dataset.prepare | 通过 |
| 项目 | project_id 异步 steps、导出/导入/trash | 通过 |
| 评测 | 同输入两次指标一致 | 通过 |
| 混音 | 2/4 轨、响度/true peak | 通过 |
| 实时 VC | 设备枚举、passthrough 30 分钟长稳 | 通过（180/180 采样，overflows=0） |
| MSVC | 3 个 .msvc 正向转换 + pth RVC 推理 | 通过 |
| UI | Edge CDP 真实页面逐一点击 12 页签 | 通过 |

## 3. 关键证据

### TTS

- 中文：job_4fdcbf57f23e47ec，3.985s，22050Hz
- 英文：job_dd9f7957e6574832，3.950s，22050Hz
- 批量：job_b0ddafed3e7f4f02 / job_c2a4f248700f4617
- 缺模型：BACKEND_PATH_NOT_FOUND
- 20001 字：INVALID_PARAMS

### MSVC

- 输入：F:\ai配音\心月-变声器\心月变声\_internal\modules\曼波Pro\苏苏.msvc
- job_d93e96a0631a4c7e，成功
- 输出：曼波Pro.pth + conversion.json
- 转换后 pth RVC：job_c8d664bdd5374942，成功，48000Hz
- 多样本：法国赌神 job_17397059fa2f4a3b、爱莉希雅 job_119f025d5f4542ef 均成功

### UI

- Edge 真实页面：标题 AMS Kernel，session 鉴权，SSE 已结束
- 12 个页签逐一点击：项目、实时变声、音色库、TTS 批量、音源炼制、数据集工作室、评测器、混音母带、人声分离、歌曲换源、MSVC 工具、TTS 语音合成
- 每个页签点击后页面均正常渲染并包含对应内容

### RVC

- 3 分钟：job_0cb4f47824fb4f80，180.000s，40000Hz，peak 0.571
- 5 分钟：job_7d4d74f0d0474d71，300.032s
- 12 分钟：job_81c3f74b2a604dfb，729.000s，3 chunks
- 无 index：job_38c3b6220efa44ce，成功
- 训练音色：job_dbcb663992eb4850，48000Hz，3.9849s

### pymss

- 3 分钟：job_1134236bfba74065，4 stems 均 180.000s，44100Hz
- 10 分钟：job_495512fb07ba42d5，4 stems 均 600.021s
- 修复：runtime.json 的 pymss_python 从嵌入式 python 改为 .venv/Scripts/python.exe

### 训练

- 闲云数据集：F:\ai歌曲\音源炼制\dataset_xianyun640（同步自 F:\AMS-Studio\data\datasets\xianyun640），538 clips，3751.2s，48k
- job_c97e6ec73cec498a，50 epoch 实际完成；任务状态因 Gateway 7200s 超时标记 failed，但最终 pth/index/voice 均已生成
- 最终音源：F:\AMS-Studio\data\voices\闲云\闲云.pth + 闲云.index（525,281,325 bytes）
- 最终 RVC：job_8b6b91e3d670423f，9.3973s，48kHz，index_used=true
- 最终评测：job_31a491ff5b8340c5，clipping=0，LUFS -17.0
- e10 checkpoint 闲云第 87 条 RVC：xianyun_e10_rvc.wav，9.3973s，48kHz，peak 0.862
- job_9fd0d6bcf75e4924，10 epoch，成功
- pth：h2_train_test.pth，57,578,929 bytes
- index：h2_train_test.index，133,466,157 bytes
- 已注册 voice；新音色 RVC 成功
- job_6f8aa6d4c0ef4e4b，50 epoch，成功
  - pth：h2_train_50.pth，57,578,011 bytes
  - index：h2_train_50.index，133,466,157 bytes
  - 新音色 RVC：job_a317bd5915a04b63，成功，48000Hz，3.9849s，index_used=true

### 项目

- 创建：p_mvnjjc8c
- ping + project_id：job_bc6bdaadf8324c09
- steps 终态：succeeded
- export/import/trash：通过

### 评测/混音

- 两次 evaluate 指标完全一致
- 4 轨 mix：192000Hz / stereo / PCM24
- LUFS：-13.8（目标 -14，误差 0.2）
- true peak：-2.1 dBFS（低于 -1）
- clipping_samples：0

### Gateway

- 取消 RVC：0.46s 进入 cancelled
- GPU：一个 running/held，一个 queued/waiting
- 多次 Gateway 重启后任务持久化可查询

### 实时 VC

- /vc/devices：VB-Cable、WASAPI loopback 均存在
- passthrough：running，frames 增长，overflows=0
- 30 分钟长稳：完成（180 个 10s 采样，running 180/180，overflows=0，frames 87,965,760）
- 跨站请求：401；无 token loopback fallback 仍为 200（本地信任设计）

## 4. 阻断/边界

1. **训练任务状态与实际产物不一致**：job_c97e6ec73cec498a 因 Gateway 7200s 超时标记 failed，但 50 epoch、最终 pth/index/voice 和 RVC 均成功；该超时/进度问题留到音源炼制专项修复。
2. **中文歌声听感**仍缺专门数据集；闲云语音用于中文语音转换测试。
3. Host 的 loopback same-origin fallback 允许无 token 访问本机 API；跨站请求已返回 401，需安全评审。

## 5. F 盘清理

已逐文件删除测试安装：

- F:\AMS-Studio-CleanTest（171 files）
- F:\AMS-Studio-H1Clean（1 file）
- F:\AMS-Studio-H1Final（5 files）

已逐文件删除死构建缓存：

- F:\_build-prod-final（3837 files，1.66GB）
- F:\_build-prod-target（3837 files，1.66GB）

保留：F:\AMS-Studio（生产）。F:\AMS-VM 已按 H3 决定彻底拆除。

## 6. 结论

核心音频链路（TTS、RVC、pymss、训练、评测、混音、项目）已通过。

H2 核心功能完成：MSVC 多容器、真实浏览器 12 页签、50 epoch 训练、闲云标准音源、最终中文 RVC/评测均通过。

唯一记录：训练任务因 Gateway 7200s 超时标记 failed，但最终产物完整可用；该问题转入音源炼制专项修复，不影响 H2 功能验收。

已部署修复后的 Gateway：`service.gateway` 本地组件已替换，mock.long 5s 正常。

## 7. 日志

- F:\AMS-Studio\data\logs\h2_core.json
- F:\AMS-Studio\data\logs\h2_rvc.json
- F:\AMS-Studio\data\logs\h2_rvc512.json
- F:\AMS-Studio\data\logs\h2_pymss.json
- F:\AMS-Studio\data\logs\h2_pymss10.json
- F:\AMS-Studio\data\logs\h2_batch_quality.json
- F:\AMS-Studio\data\logs\h2_gpu_serial.json
- F:\AMS-Studio\data\logs\vc_30min.json
- F:\AMS-Studio\data\logs\h2_msvc.json
- F:\AMS-Studio\data\logs\h2_msvc_rvc.json
- F:\AMS-Studio\data\logs\h2_msvc_multi.json
- F:\ai歌曲\音源炼制\h2_final_xianyun_rvc.json
- F:\ai歌曲\音源炼制\h2_final_xianyun_eval.json
- F:\AMS-Studio\data\datasets\ys6.3cn_xianyun_manifest.json（闲云 640 条）
