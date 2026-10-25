# AMS Studio 项目完整记录（2026-10-25）

> 本文是项目唯一总记录，合并阶段 A–G 的执行记录、设计决策、测试证据、当前状态、发布与清理结果。历史阶段文件只作为归档，不再作为入口。

## 1. 项目定位

AMS Studio 是本地优先的 Windows AI 音乐工作站：

- Tauri 启动器；
- DeepSeek Harness Kernel；
- Cordis 音乐插件；
- FastAPI Gateway；
- RVC/pymss/Piper/MSVC 本地能力；
- 所有用户数据位于安装目录 data；
- 不依赖云端账户，不使用用户 C 盘 AppData 存 AMS 数据。

## 2. 仓库结构

| 目录 | GitHub | 作用 |
|---|---|---|
| launcher | yan-mc/ams-launcher | Tauri 启动器、组件安装、Kernel 生命周期 |
| kernel | yan-mc/ams-harness | DSH fork、插件宿主、Kernel |
| plugins | yan-mc/ams-plugins | 音乐 UI + Host 鉴权桥接 |
| services | yan-mc/ams-services | Gateway、RVC/pymss/TTS/train/evaluate/mix |
| tools | yan-mc/ams-tools | MSVC/模型转换工具 |
| docs | yan-mc/ams-docs | 设计、计划、发布、事故记录 |
| release | yan-mc/ams-release | feed、签名、打包、发布说明 |
| 总仓库 | yan-mc/AMS-Studio | submodule 指针与总说明 |

本地源码根：F:/Deepseek_harness/AI音乐处理/08_项目代码  
本地安装根：F:/AMS-Studio  
本地 data：F:/AMS-Studio/data

## 3. 阶段结果

### A 基础修复

- data 布局、文件选择、上传、文件夹按钮、MSVC 独立、说明书；
- 数据缓存/临时目录收口到 data；
- 完成度 95%。

### B 实时变声器

- 独立 VC 服务；
- 麦克风、WASAPI loopback、系统输出；
- 麦克风 + loopback 混音；
- RVC/passthrough、VB-Cable、耳机监听；
- 48k split stream、WASAPI 重试、电平/p95、极低延迟档；
- 完成度 96%；外部 p95/长稳未完成。

### C 参数与音源

- RVC 全参数；pymss 参数；
- 模板保存/导入/导出；
- 音色库标签/备注/收藏、index 绑定/解绑、试听；
- voicepack 导入/导出；
- TTS 长文本、任务试听、schema；
- 完成度 92%；TTS 只有单模型的历史限制已由双模型补充。

### D 音源炼制

- dataset.prepare：导入/切片/校验；
- train.rvc：预处理、F0、HuBERT、train、FAISS index、注册；
- v2/48k、v1/40k、f0=0、取消；
- LibriSpeech 8.08 分钟、LJSpeech 14.8 分钟验证；
- 完成度 95%；中文歌声质量未验。

### E 歌曲换源

- 分离结果一键回填；
- A/B、批量、逐项重试；
- 纯人声导出；
- 12:09 长音频自动 300 秒分段 + 50ms 交叉淡化；
- 长度锁定、爆音保护、RMS/peak；
- 完成度 97%。

### F 项目/评测/混音

- project.json、项目 UI、步骤、导入导出；
- evaluate.audio：时长/采样率/声道/削波/静音/RMS/SNR/LUFS/音高偏差；
- mix.master：多轨、逐轨增益/声像/三段 EQ/HP/LP/静音、总线压缩、loudnorm/alimiter；
- 完成度 90%；插件未拆，混音不是完整 DAW。

### G 发布

- feed 2042；
- setup 2,438,343 字节；
- 原地升级验证保留 data；
- 测试工程移入 trash；
- 完成度约 70%；干净 VM、卸载、常驻服务仍未完全验收。

## 4. 关键测试证据

- RVC 3 分钟换源：job_8652dada37e04bea，RTF 0.0412。
- 12:09 长音频：3 段，输出 729.0 秒，RTF 0.0313，VRAM 2560MB。
- LJSpeech 训练：job_1947d953aa3e471a，10 epoch，pth+index。
- 数据集准备：job_173d515c33c447da，133 片/889 秒。
- TTS 双模型：job_e5971623cb424566 / job_c4b7145decf44b51，均成功。
- 项目步骤：p_mvne99ku，evaluate job_8aa6779abb25405a，steps=1 succeeded。
- 浏览器 CDP：项目/评测器/混音/换源/数据集/训练页全部通过。

## 5. 数据与安全边界

- 用户音色只有曼波；
- 测试工程移入 data/trash/projects；
- 测试数据集仍保留在 data/datasets：d0_smoke、d0_prepared、d0_gateway、libri422、ljspeech15；
- 不删除用户文件；
- 禁止 C 盘用户目录、Program Files、Windows、ProgramData、盘根递归删除；
- 删除先确认路径并优先移动到隔离目录；
- 根目录阶段备份/日志归档，不在源码仓库提交。

## 6. 当前发布状态

- feed：2042；
- setup：2,438,343 字节，SHA256 D2DEFF5125CD857F41D9CE7A7C689E79A1DDA88AADA55F486DF4AE6060338B45；
- 已做原地升级，data 保留；
- 干净 VM/卸载实测未完成；
- Kernel watchdog 当前会话运行，不是系统服务。

## 7. 已知限制

1. 干净 VM/卸载未完全验收；
2. Kernel 无系统级常驻；
3. F 插件未拆；
4. 混音不是完整 DAW；
5. VC 长稳未做；
6. 测试数据占用大；
7. feed CDN 有延迟；
8. 部分自动化只覆盖页签/表单，不覆盖真实播放。

## 8. 归档与清理原则

- 旧阶段日志、测试备份、构建中间产物移入归档目录；
- 用户音色、真实项目、模型权重不移动；
- 不使用目录递归删除；
- 所有清理保留清单和日期；
- 发布 feed 与 setup 必须一起记录。

## 9. 下一步

- 完整干净 VM 安装/卸载；
- Kernel 系统级常驻策略；
- F 插件拆分；
- 混音完整母带链；
- VC 长稳压测；
- 数据清理自动化策略。
