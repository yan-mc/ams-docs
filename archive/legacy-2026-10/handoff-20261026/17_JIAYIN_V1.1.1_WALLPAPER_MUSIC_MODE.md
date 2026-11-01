# 17 · 嘉音Studio v1.1.1：安装体验、壁纸与「音乐制作」模式

- 日期：2026-10-31（Asia/Shanghai）
- 版本：launcher/setup **1.1.1**；agent-music **0.2.2**；feed **2056**；kernel 0.2.1-rc.3；gateway/plugin 1.0.0
- 发布：`yan-mc/ams-release` **v1.1.1（latest，release id 429789886）**；`yan-mc/ams-agent-music` **v0.2.2（id 429788826）**

## 1. 本轮目标（用户 2026-10-31 指令）

1. 内核任务栏图标**不再处理**（保持插件不改内核 exe）。
2. 启动器任务栏图标换成正确的新 logo。
3. 安装时在**桌面创建「嘉音Studio」快捷方式**（并配套开始菜单/卸载项）。
4. agent-music 新增**换壁纸**功能，壁纸用桌面 `FgAGOzumea1BaCuv5tpKcU5oLhRO.webp`。
5. agent-music 新增**「音乐制作」模式**（标准/创造之外），默认加载音乐 Skill/工具，默认工作目录 = 内核所在 `data` 文件夹，**不得写死路径**。

## 2. 实现

### 2.1 启动器（v1.1.1）

- 图标：`src-tauri/icons/*` + `src/assets/logo.png` 已是新圆形 logo；1.1.1 重新构建后 exe 内嵌图标为新 logo（提取 32×32 校验）。窗口/任务栏图标随 exe 资源。
- 快捷方式（NSIS `ams-setup.nsi`）：
  - 安装：`$DESKTOP\嘉音Studio.lnk`、`$SMPROGRAMS\嘉音Studio\嘉音Studio.lnk`、`卸载 嘉音Studio.lnk`，图标均为安装目录 `ams-launcher.exe`。
  - 卸载：删除上述快捷方式与开始菜单目录，保留 `data`。
- 一键安装插件：`install_music_plugins` 前缀列表加入 `ams-agent-music-mode-`；模块校验加入 `agent-music-mode`；组件状态「Agent 插件」同步要求 mode 模块，详情显示「20 tools + Skill + 内核换标 + 壁纸 + 音乐制作模式」。
- 两个说明面板新增：Agent 插件功能新增「五、壁纸与音乐制作模式」；软件功能总览新增「16. 内核换标、壁纸与音乐制作模式」。版本号 1.1.1。

### 2.2 壁纸（@ams/agent-music-ui 0.2.2）

- 桌面原图 `FgAGOzumea1BaCuv5tpKcU5oLhRO.webp`（3800×2137，1,074,032 B）→ Pillow 压缩为 1600×900 JPEG（256,076 B）→ base64 data URI（341,459 字符）内联进 client bundle（client tgz 约 294KB）。
- 运行时只做 CSS 注入（不写文件、不改内核）：
  - `html, body` 背景透明；`body::before` 固定全屏壁纸（cover/center，`z-index:-2`）；`body::after` 深色渐变遮罩（42%→66%）保证可读；
  - 主框架/侧栏按 class 片段半透明覆盖（`_2jj0tW_frame`/`NpfQ0a_root`/`_emptyTabHost`/`_2jj0tW_sidebarCol`/`_8aqHxG_root`），品牌区 `kdKC2a_root` 透明；
  - MutationObserver/定时兜底重注入，SPA 重渲染后仍生效。
- 壁纸来源文件同时留档在 `_接管审计/brand/wallpaper-1600.jpg` 等（原始 webp 不入库）。

### 2.3 音乐制作模式（preset + 自动工作目录）

- 新 preset 行（bundle patch）：`@deepseek-ai/dsh-agent-preset`，`id: ams-music`，`name: 音乐制作`，`order: 2`，与 standard/creator 并列。
- preset 子插件：persona（音乐制作 Prompt）、agent-instructions、time-context、tool-pwsh/fs/fs-search、tool-jobs/schedule/todo/ask-user/web/present、skill-filesystem + tool-skill（可调用 `ams-music-processing` Skill）、command-goal/tool-goal、compaction 组、以及新增 `@ams/agent-music-mode`。
- `@ams/agent-music-mode`（新增包）：监听 `agent/created`，解析 `AMS_HOME`（→ Kernel cwd 的父目录 → `DSH_HOME` → 旧默认），若存在则调用 `ctx.workingDirectory.set(agent, dataDir)`，即**新会话默认工作目录 = 安装目录 data**；`workingDirectory` 为会话级服务，改为运行期 `ctx.get()` 惰性获取，避免 preset 提前挂载时 inject 失败。
- 兼容性修复：preset 首版因 `tool-todo` 缺少必填 `config.allowParallelInProgress` 而进入 broken 名单；补 `allowParallelInProgress: true` 后 `agentPresets/list` 无 broken，UI 菜单正常显示并可选。

## 3. 验证证据

- 启动器 WebView2 CDP（`_接管审计/qa/v1.1.1-20261031/`）：
  - `launcher-main.png`：标题/`h1=嘉音Studio`、新圆形 logo（naturalWidth=256）、版本 v1.1.1、三个按钮；
  - `launcher-agent-guide.png`：面板标题「Agent 插件功能（20 个工具 + 壁纸 + 音乐制作模式）」且含「壁纸」「音乐制作模式」「五、壁纸」；
  - `launcher-feature-guide.png`：含「内核换标」「壁纸」「音乐制作模式」「17. 已知限制」。
- 安装器（静默安装到临时目录 app3）：桌面快捷方式 `C:\Users\28988\Desktop\嘉音Studio.lnk` → `F:\AMS-Studio\ams-launcher.exe`，开始菜单 `%APPDATA%\...\嘉音Studio\{嘉音Studio.lnk, 卸载 嘉音Studio.lnk}`；安装 exe 版本 1.1.1；提取图标为新 logo。
- 内核 CDP（用户安装 `F:/AMS-Studio`，agent 0.2.2 dev 安装）：
  - `kernel-wallpaper.png`：壁纸可见、UI 半透明、文字可读；
  - `kernel-music-mode-menu.png`：预设菜单三项「标准模式 / 音乐制作 / 创造模式」，音乐制作带说明；
  - `kernel-music-mode-selected.png`：选中后 seat label = `音乐制作`；
  - `agentPresets/list`：`ams-music` 存在且无 `broken` 字段。
- 模式插件单测（Node）：`resolveAmsHome()` → `F:\AMS-Studio\data`；模拟 `agent/created` → 调用 `workingDirectory.set(agent, F:\AMS-Studio\data)` → PASS。
- 全新安装（feed 2056）：安装器 → `--install-components` 从签名 feed 下载全部组件（含 `plugin.ams.agent.music 0.2.2`）→ 启动内核初始化 → `--install-plugins` 安装 4 个 tgz（host/ui/bundle/mode）→ 组件状态「Agent 插件 已安装到内核」。

## 4. 发布

| 项 | 值 |
|---|---|
| ams-release latest | v1.1.1（id 429789886），feed **2056**（14 组件） |
| setup | `AMS-Studio-Setup-1.1.1.exe` 3,439,487 B / sha256 `0BBF5C86…D924` |
| feed | `ams-components.json` 10,326 B / sha256 `71852005…E255` |
| manifest | `RELEASE_MANIFEST.json` 767 B / sha256 `DAD0DAA4…65F1` |
| agent-music | v0.2.2（id 429788826），zip 657,100 B / sha256 `661A65CD…545AC` |
| commits | launcher `d92102c`；agent-music `fa655f8` |
| 本地物料 | `09_发布/当前/V1.1_1.1.1_feed2056/` |

v1.1.0 已改为 previous stable（名称/正文更新，资产保留）。

**用户安装同步升级（2026-10-31 15:50）**：`F:\AMS-Studio` 已覆盖安装 v1.1.1；Agent 组件从 GitHub v0.2.2 资产下载并解压到 `data/components/plugin.ams.agent.music`，`--install-plugins` 成功后 profile 依赖全部指向组件托管 tgz；Kernel 重启后日志 `plugin_version 0.2.2`、`tools-registered count=20`，组件状态「音乐插件/Agent 插件 已安装到内核」，桌面快捷方式目标正确。

## 5. 已知限制（未变）

- **Kernel exe/任务栏图标无法由插件修改**（用户已确认不再处理）；页面内 logo/标题/壁纸可改。
- ASR 的 Vosk 依赖与中文模型仍不在 feed（可选自备/旧数据恢复）。
- 20 tools 自动化回归、ASetup Authenticode 签名、GitHub 大文件下载续传仍为后续项。
- 「音乐制作」preset 的子插件若在 Kernel 升级后变更 Config 必填项，可能出现 broken（诊断可从 `agentPresets/list` 的 `broken` 字段读出）；本轮已按 rc.2 实际配置校验。

## 6. 文件索引

- launcher：`src-tauri/src/lib.rs`（mode 包/状态）、`installer/ams-setup.nsi`（快捷方式）、`src/App.vue`（说明面板）
- agent-music：`client/src/client.template.js`（壁纸注入）、`mode/`（新包）、`bundle/cordis.patch.yml`（music preset）、`build.mjs`、`package.py`
- 证据：`_接管审计/qa/v1.1.1-20261031/`
- 发布：`09_发布/当前/V1.1_1.1.1_feed2056/`
