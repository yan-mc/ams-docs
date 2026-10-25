# H2 核心功能回归结果（2026-10-25）

状态：**核心音频链路通过；H2 仍有环境/质量边界，不能无条件签收。**

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
| MSVC | 无样本，负向错误码 | 阻断：无 .msvc 样本 |
| UI | 页签静态检查 | 通过（无浏览器交互自动化） |

## 3. 关键证据

### TTS

- 中文：job_4fdcbf57f23e47ec，3.985s，22050Hz
- 英文：job_dd9f7957e6574832，3.950s，22050Hz
- 批量：job_b0ddafed3e7f4f02 / job_c2a4f248700f4617
- 缺模型：BACKEND_PATH_NOT_FOUND
- 20001 字：INVALID_PARAMS

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

1. **MSVC 无真实 .msvc 样本**，只验证了负向 INPUT_PATH_MISSING；不能签收转换成功。
2. **中文歌声数据集缺失**，无法完成中文歌声听感验收；50 epoch 功能链路已通过。
3. **UI 只做页签静态检查**，没有浏览器级交互/视觉自动化。
4. mock.long 在打包 Gateway 中用 sys.executable 启动 Python，子进程 exit 2；源码已修复，但当前已安装 Gateway 未重建。
5. Host 的 loopback same-origin fallback 允许无 token 访问本机 API；跨站请求已返回 401，需安全评审。

## 5. 结论

核心音频链路（TTS、RVC、pymss、训练、评测、混音、项目）已通过。

H2 暂不能无条件签收，需完成：

- MSVC 真实样本；
- 中文歌声数据集/听感；
- UI 浏览器级验收；
- 重建并部署修复后的 Gateway。

## 6. 日志

- F:\AMS-Studio\data\logs\h2_core.json
- F:\AMS-Studio\data\logs\h2_rvc.json
- F:\AMS-Studio\data\logs\h2_rvc512.json
- F:\AMS-Studio\data\logs\h2_pymss.json
- F:\AMS-Studio\data\logs\h2_pymss10.json
- F:\AMS-Studio\data\logs\h2_batch_quality.json
- F:\AMS-Studio\data\logs\h2_gpu_serial.json
- F:\AMS-Studio\data\logs\vc_30min.json（完成后生成）
