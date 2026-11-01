# 05_新方案探讨 —— 新时代开源 AI 音乐制作套件（可行性阶段）

> 2026-10-19 第二阶段产出。回答“启动器 + DSH 插件化主体 + 四个功能插件 + V 家式填词”是否可行、缺什么、怎么做最稳。
> 本目录是**可行性讨论**，不是完整实施方案。

## 先读这个

| 文件 | 内容 |
|---|---|
| **可行性探讨.md** | 主文档：总体判断、架构、DSH 承载、四个插件、音源格式、填词成歌、缺口、风险、验证实验、待决策问题 |
| 01_心月格式深拆/报告.md + 格式.md | 心月 MSVC 模型格式逐字节规范；已实测转回标准 RVC .pth 并推理成功；.cyd 加密结论 |
| 02_SVS填词/报告.md | 2026 开源 SVS 全景（DiffSinger/OpenUtau/SoulX/YingMusic/ACE-Step/YuE2）、三条管线、缺口与许可 |
| 03_开源栈调研/报告.md + voice_pack.schema.json | 2026-10 VC/实时变声/TTS/训练/分离选型、许可矩阵、统一音源包 UVP v0.1 |
| 04_DSH架构/报告.md + 清单.json | DSH 插件/技能/页面/任务/打包/启动器评估、四类可行性清单、Spike 清单 |

## 五个核心结论（详见主文档）

1. **心月格式已拆开**：它没有改模型架构，只是把标准 RVC v2 权重封进自研容器（头 + 3 段反转 payload + 分片重排 + 机器指纹/身份牌 + 授权校验）；权重是明文 FP16。已成功还原 457 张量、生成 `.pth`、用 RVC runtime 加载并推理（unexpected=0，missing 全是 enc_q，正常）。但 `.cyd`/密码学模块静态无法解密（自锁，需动态调试），新方案**不应依赖**。
2. **DSH 路线可行，但要认清边界**：MIT、Cordis「万物皆插件」，可用自定义 profile/bundle 组合出“只保留核心 + 音乐插件”的发行版；Host 插件能注册 tool/skill/jobs/subprocess/本地 HTTP，Client 插件能加全页面/导航/设置。边界是：不能替换 DSH 外壳（要独立 DAW 壳需 fork 或 SDK+自研前端）、客户端构建链未发布、白标/裁剪/签名/更新需 fork 构建、插件无沙箱、默认遥测要关。
3. **四个插件的主后端已明确**：分离 pymss（MIT），换声/训练 RVC 官方 2.3.260718（MIT），变声 RVC 实时 GUI + 官方 VST2/VST3，TTS 用 GPT-SoVITS v2Pro（中文微调）+ Qwen3-TTS（Apache 零样本）；Seed-VC/so-vits-svc 已归档，不应作为必选。
4. **填词成歌比预期乐观**：DiffSinger/OpenUtau 做主路径（需自研无头渲染器 + 音节分配 + 编辑器，核心约 2–4 人周），SoulX-Singer（Apache-2.0 零样本 SVS）和 YingMusic-Singer-Plus（CC BY 4.0 保旋律改词）做加速/兜底；正确顺序是 SVS 干声 → SVC/RVC。
5. **必须补的缺口**：数据集工作室、音色包/模型库、混音母带、评测器、项目管理、实时音频路由、多后端环境管理、更新回滚、许可合规。

## 建议的分期（不是完整方案）

- **阶段一（4–8 周）**：官方 DSH 桌面版 + 自定义 music bundle（Host 工具/jobs + Client 整页 + skills），验证“训练任务管理 / 音频编辑器 / 模型下载”三条主线；
- **阶段二（6–12 周）**：fork DSH 做白标/裁剪 + 独立 Tauri 启动器；DSH 作为本地 agent 内核；
- 先做 Spike：DSH 最小插件闭环、DiffSinger 端到端、SoulX/YingMusic 试听、pymss A/B、RVC 实时延迟、UVP 双后端包。

## 只读说明

- 本阶段未修改源目录（F:\ai歌曲、F:\ai配音、D:\dsh、AppData）；所有写入在本目录。
- 心月格式转换样本仅用于技术验证，**不扩散权重**；不建议把其模型/解密代码打包进新产品。
