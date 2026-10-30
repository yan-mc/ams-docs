# 16. 嘉音Studio v1.1 改版 + Agent 插件 0.2.1（2026-10-31）

范围：需求全量落地（品牌改版、启动器说明面板、Agent 插件修复/一键安装/内核换标、发布 v1.1.0）。
发布：**嘉音Studio v1.1.0 / feed 2055 / GitHub latest**（release id 429294171）。
源码：launcher `fe3c190`、agent-music `0bd5641`；agent-music release v0.2.1（id 429282180）。

## 1. agent-music 修复（v0.2.1）

| 问题 | 修复 |
|---|---|
| 审计日志硬编码 `F:/AMS-Studio/data/logs/...` | 新增 `resolveAmsHome()`：AMS_HOME → Kernel cwd(`<home>/kernel`) → DSH_HOME → 最后回退；日志落 `<AMS_HOME>/logs/agent-music-load.log`。实测 Kernel 自动得到 `F:\AMS-Studio\data` |
| ASR 依赖路径与实际 runtime 布局不符（`runtime/tts-train/0.1.0/runtime.tts-train-0.1.0/python`） | 自动探测平坦/嵌套两种布局，扫描 tts-train 子目录；deps/model/ffmpeg 同样自动发现；缺依赖时返回明确的 `ASR_*_NOT_FOUND` 说明（含可恢复路径提示）；保留环境变量覆盖 |
| `engines.dsh` 声明 rc.2 | 保持 0.2.1-rc.2（与 Kernel 内部版本一致；rc.3 会被插件管理器拒绝，实测确认） |
| 无内核换标 | 新增 **@ams/agent-music-ui** Client 插件（运行时 DOM 注入，不改内核源码） |

构建：`build.mjs` 同时产出 Host lib 与 Client bundle（logo 以 data URI 内联）；`package.py` 产出
raw 包目录 + `tgz/`（host/ui/bundle 三个 npm 风格包）+ zip；发布 zip 131,221 B，sha256 `36b876f7…0aab`。

## 2. 内核换标（纯插件实现，未改 Kernel）

实现方式：`@ams/agent-music-ui` 通过 DSH Client 插件机制加载进 Kernel Web 页面，运行时执行：

- `document.title` → 嘉音Studio（含 3 秒自愈，防路由重置）；
- `sidebar.brand.mark` 与 `conversation.hero.brand.mark` 的鲸鱼 SVG（viewBox `0 0 23.16 17.04`）→ 新圆形 logo（data URI，24/34px）；
- `sidebar.brand.name` 的 deepseek/HARNESS 文字 SVG 路径 → `<span>嘉音Studio</span>`；
- 文本节点 `探索未至之境 / DeepSeek / deepseek / HARNESS / Harness` → 嘉音Studio；
- favicon link → 新 logo；MutationObserver + 定时兜底应对 SPA 重渲染。

实测（CDP 截图 + DOM 调查）：窗口/页面标题=嘉音Studio；左上 logo+嘉音Studio；正中间 logo+嘉音Studio+预览版；`sidebar.brand.*` / `conversation.hero.brand.mark` 均为新内容；AMS 音乐页/侧栏功能不受影响。
Host 插件同时正常加载：`tools-registered count=20`、`skill-list ams-music-processing`、`selftest ok=true`、数据根=当前安装。

**边界（插件做不到）**：Kernel 可执行文件（AMS Kernel.exe）的任务栏/文件图标属于 exe 资源，插件无法修改；需要重编内核 exe（未改内核源码）。启动加载页的 HARNESS 字样如出现也在运行时被替换。

## 3. 启动器改版（v1.1.0）

- 产品名 **嘉音Studio**：窗口标题、App 头部、安装器 Name/Caption/BrandingText/DisplayName、卸载按钮；
- 图标：桌面 `嘉音Studio风格logo设计.png` → Pillow 处理（按圆形内容 bbox 裁切、圆形 alpha 遮罩抗锯齿、白边透明、豆包水印在圆外被裁掉），生成 16–1024 PNG + 多尺寸 ICO；
- 头部 logo + 副标题；`document.title` 同步（实测 WebView2 标题=嘉音Studio）；
- NSIS 增加 `/INPUTCHARSET UTF8`，修复中文 DisplayName 乱码（实测注册表=嘉音Studio）；
- 新增两个详细说明面板（按钮与“使用说明”并列）：
  * **Agent 插件功能**：20 个工具分组、Skill、ASR 依赖、安全边界、6 个对话示例；
  * **软件功能总览**：产品定位 + 14 个音乐页签逐项用法（项目/音源炼制/换源/实时变声/音色库/人声分离/混音母带/数据集/评测/TTS 合成/批量/TTS 数据集/TTS 音源炼制/MSVC/任务系统）+ 本地优先/隐私 + 硬件与快速上手 + 已知限制；
  * 旧“使用说明”同步更新（去掉“当前未实现”的过时描述）。

## 4. Agent 插件一键安装（启动器）

- feed 增加组件 `plugin.ams.agent.music` 0.2.1（zip 内含 host/ui/bundle 三个 tgz）；
- `install_music_plugins` 扩展：递归发现 Agent tgz（前缀+版本号数字校验，避免 `ams-agent-music-*` 误匹配 bundle/ui），依次 `dsh plugin add`，校验 `@ams/{agent-music,agent-music-ui,agent-music-bundle}`；
- 按钮改为“安装插件到内核（音乐 + Agent）”；新增 CLI `--install-plugins`；
- `component_status` 新增“Agent 插件”行（未安装 / 已下载（未安装到内核）/ 已安装到内核）。

实测（干净目录 + 本地测试 feed）：安装 setup 1.1.0 → 安装 kernel+音乐+Agent 组件（downloads 0）→ 启动内核一次初始化 profile → `--install-plugins` 成功（6 个 @ams 模块）→ `--component-status` 显示音乐插件/Agent 插件均“已安装到内核”。

## 5. 发布

| 项 | 值 |
|---|---|
| GitHub release | 嘉音Studio v1.1.0（id 429294171，prerelease=false，latest） |
| feed | **2055**（14 组件：原 13 + plugin.ams.agent.music），sha256 `43AD9863…82D1`，验签 OK |
| setup | `AMS-Studio-Setup-1.1.0.exe`，3,438,229 B，sha256 `1DCE2F79…E0BA` |
| launcher | 7,228,416 B，sha256 `6019F40E…07A6`，6 个内嵌前端资源 |
| agent-music | v0.2.1（独立 repo release，zip 131,221 B / `36B876F7…0AAB`） |
| 本地物料 | `09_发布/当前/V1.1_1.1.0_feed2055/` |
| 旧版本 | v1.0.2/v1.0.1/v1.0.0 = previous stable；v0.2.1 = older stable；rc.1/rc.2 = superseded |

发布后验证：latest=v1.1.0；latest feed seq=2055、14 组件、验签 OK；5 资产 digest 一致。

## 6. 用户操作建议

1. 关闭当前启动器 + 内核；
2. 运行 v1.1.0 setup 覆盖安装（数据不动）；
3. 启动启动器 → “检查组件清单 / 下载并安装全部”（会新增 Agent 插件组件）→ 启动内核一次 → 停止 → “安装插件到内核（音乐 + Agent）” → 重启内核；
4. 之后 Kernel 界面即为嘉音Studio；启动器有三个说明按钮；Agent 在对话中可直接操作音乐流程。
