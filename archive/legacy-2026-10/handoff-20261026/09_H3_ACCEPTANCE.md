# H3 验收报告（UI CDP + GPU E2E）

日期：2026-10-26（Asia/Shanghai）  
执行：接手负责人（Lead）+ 团队成员  
被测对象：AMS Studio H3 1.0.0-rc.1（离线恢复环境）+ rc.2 修复组件  
环境根：`F:/AMS-H4-H3`（setup 1.0.0-rc.1 安装 + 本地资产离线恢复）  
证据目录：`_接管审计/qa/`

---

## 0. 结论

| 验收项 | 结果 | 证据 |
|---|---|---|
| UI 14 页签 CDP（H3 实际口径） | **14/14 PASS，0 FAIL，0 WARN** | `qa/ui-cdp-20261026-163803/`（results.json + summary.md + 14 张截图） |
| GPU E2E 基础 6 项 | **6/6 PASS** | `qa/gpu-e2e-20261026-163702/` |
| GPU E2E 扩展（12 分钟长音频 + 1 epoch 训练） | 执行中/结果见 `qa/gpu-e2e-20261026-1638xx/` | 见 §3.3 |
| Kernel 认证 | 无 `Web authentication failed` | Kernel Web `http://127.0.0.1:1279` HTTP 200，title=AMS Kernel |
| 20 tools + skill | PASS | `data/logs/agent-music-load.log`：tools-registered count=20、skill-list ams-music-processing、skill-content bytes=3182、selftest ok=true |
| Gateway | PASS | `/api/v1/health` 200 ok=true；job_types 含全部 12 类（含 mix.master） |

> 备注：H4 旧文档写"12 页签"，H3 1.0.0-rc.1 实际为 **14 页签**（新增 TTS 数据集、TTS 音源炼制）。

---

## 1. 环境

| 项 | 值 |
|---|---|
| 安装根 | `F:/AMS-H4-H3` |
| launcher | 1.0.0-rc.1（setup 安装），后由 rc.2 setup 验证另见 H4 报告 |
| Kernel | `data/kernel`（ams-kernel-installlocal-0.2.1-rc.3-final，sha `CE507367…`），Web 1279 |
| Gateway | rc.2 修复版（mix.master 恢复；`A82916DC…`），动态端口（E2E 时 7279） |
| 插件 | @ams/plugin-core 1.0.0-rc.1 + @ams/plugin-ui 1.0.0-rc.1 + @ams/music-bundle 1.0.0-rc.1 + @ams/agent-music 0.2.0 |
| 数据 | voices 曼波/闲云/xianyun_zh、datasets xianyun640/tts、tts_voices、projects imports（e_audit_3min/12min） |
| runtime | rvc 0.1.0（10.37GB）、pymss 0.1.0、tts 0.1.0、tts-train 0.1.0（6.62GB）、pymss.python、tool.msvc、tool.vcrun、service.vc（vc-service） |
| GPU | RTX 4080 Laptop 12GB（可用约 10GB），驱动 621.01 |
| 系统 | Windows 11 26H2 Build 26300.9711 |

Gateway 启动时注入的环境变量（关键，runner 只读环境变量）：

```
AMS_HOME=F:/AMS-H4-H3/data
AMS_RVC_PYTHON/AMS_RVC_ROOT      <- data/config/runtime.json
AMS_PYMSS_PYTHON/AMS_PYMSS_ROOT  <- data/config/runtime.json
AMS_MSVC_TOOL                    <- data/config/runtime.json
AMS_TTS_ROOT=F:/AMS-H4-H3/data/runtime/tts/0.1.0
AMS_TTS_TRAIN_ROOT=F:/AMS-H4-H3/data/runtime/tts-train/0.1.0
```

---

## 2. UI CDP 验收（14 页签）

脚本：`08_项目代码/ams-release/qa/ui-cdp.ps1`  
方法：Edge CDP（127.0.0.1:9223）→ 打开 Kernel → 关闭预览弹窗 → 点击侧栏「AMS 音乐」→ 逐页签点击/断言/截屏/收集 console 错误。

命令：

```powershell
pwsh -File qa/ui-cdp.ps1 -KernelUrl 'http://127.0.0.1:1279/?token=<token>' -CdpPort 9223
```

结果（`qa/ui-cdp-20261026-163803/summary.md`）：

| # | 页签 | 结果 |
|---|---|---|
| 1 | 项目 | PASS |
| 2 | 实时变声 | PASS（部署 service.vc 后 502 消除） |
| 3 | 音色库 | PASS |
| 4 | TTS 批量 | PASS |
| 5 | 音源炼制 | PASS |
| 6 | 数据集工作室 | PASS |
| 7 | 评测器 | PASS |
| 8 | 混音母带 | PASS |
| 9 | 人声分离 | PASS |
| 10 | 歌曲换源 | PASS |
| 11 | MSVC 工具 | PASS |
| 12 | TTS 语音合成 | PASS |
| 13 | TTS 数据集 | PASS |
| 14 | TTS 音源炼制 | PASS |

- 每个页签均有截图：`qa/ui-cdp-20261026-163803/NN-<页签>.png`
- 全程无 console error / exception（首次运行时"实时变声"因 VC 服务未部署出现 502，部署 `service.vc` 到 `data/components/vc-service` 后复测为 0 error）。

---

## 3. GPU E2E 验收

脚本：`08_项目代码/ams-release/qa/h4-gpu-e2e.ps1`  
方法：通过 Gateway REST API 提交真实任务，轮询到终态，读取 `/jobs/{id}/artifacts` 并校验产物（文件存在、大小/stems 数量等）。

### 3.1 基础 6 项（`qa/gpu-e2e-20261026-163702/`，FAIL=0）

| 用例 | job id | 结果 | 产物 |
|---|---|---|---|
| ping | job_0ae3130d6e684598 | PASS | 1 |
| convert.rvc 3min（曼波，cuda:0） | job_52f14f34af2d4e90 | PASS（26s） | 2（wav+report） |
| separate.pymss 3min（scnet_checkpoint_musdb18） | job_0c1860ed2dae4236 | PASS | 5（4 stems+report） |
| tts.synthesize 中文（huayan） | job_80172ad481704d08 | PASS | 1 |
| evaluate.audio 3min | job_58afd794144a473c | PASS | 1 |
| mix.master 2 轨（-14 LUFS / -1 dBTP） | job_95dfe95b009f4fe6 | PASS | 1 |

### 3.2 产物位置

所有产物在 `F:/AMS-H4-H3/data/artifacts/<job_id>/`，Gateway 同时写入 `data/logs/gateway/gateway.log` 与 job 状态。

### 3.3 扩展项（进行中/另行记录）

- `convert.rvc` 12 分钟长音频（分段 + 长度锁定）
- `train.rvc` 1 epoch（xianyun640 processed，用于验证训练→注册→可用链路）

结果以 `qa/gpu-e2e-20261026-1638xx/results.json` 为准。

---

## 4. 验收中发现的代码问题（已修复并推送）

| 问题 | 影响 | 修复 |
|---|---|---|
| Gateway `mix.master` runner 未注册（TTS 提交重写 registry 时丢失） | H4 混音链路不可用；`/health` job_types 缺 mix.master | `ams-services 2f00523`；`gateway pytest 41 passed`；重建 gateway 后 job_types 12/12 |
| H3 已发布 setup 内 launcher 为 dev 模式 cargo build（无内嵌前端资源） | 干净环境 launcher 白屏、依赖 localhost:1420；线上 setup 不可逐字节复现 | `ams-launcher efafe9f`：改用 `pnpm tauri build`，build-installer 增加"必须内嵌前端资源"断言；rc.2 setup 重建 |
| QA 脚本 artifact 解析与 ports.json 结构不匹配 | E2E 误报 FAIL | `ams-release 9c07c30`：解析 `{ok,data}` 信封与 `gateway.port` |
| UI CDP 未处理预览弹窗/侧栏入口 | 首次运行 14 页签全部 missing | `ams-release 9c07c30`：前置关闭弹窗 + 点击「AMS 音乐」 |

---

## 5. 已知边界

1. 本报告环境是"离线恢复"环境（本地组件 + HF runtime），不是严格 feed 全量安装；严格安装验证见 H4 报告。
2. 实时变声只验证页面与 VC 服务健康（无真实麦克风/长稳回归；H2 已做 30 分钟 passthrough 长稳）。
3. TTS 训练质量仍封存（TTS_ALIGNMENT_BUG 已修复音素 pad，但听感未继续调优）。
4. 中文歌声训练听感未验收（数据集限制）。

---

## 6. 复现命令

```powershell
$amsHome = 'F:/AMS-H4-H3/data'
# 1) 启动 Gateway（带 runtime env，详见 _接管审计/05_环境恢复报告.md §9.2）
# 2) 启动 Kernel（详见同报告 §9.3），记下 data/runtime/kernel.log 中的 dsh web URL
# 3) UI CDP
pwsh -File '08_项目代码/ams-release/qa/ui-cdp.ps1' -KernelUrl '<kernel url>' -CdpPort 9223
# 4) GPU E2E
pwsh -File '08_项目代码/ams-release/qa/h4-gpu-e2e.ps1' -AmsHome $amsHome
# 5) 扩展 E2E
pwsh -File '08_项目代码/ams-release/qa/h4-gpu-e2e.ps1' -AmsHome $amsHome -IncludeTrain -IncludeLong
```
