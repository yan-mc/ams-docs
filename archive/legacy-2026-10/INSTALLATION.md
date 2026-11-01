# AMS Studio 完整安装与使用说明书

> 适用版本：AMS Studio Launcher 0.2.0 · 公开组件 feed 2042 · 音乐插件 0.3.0 · Gateway 0.1.0-stageG2  
> 适用系统：Windows 10 / 11 x64  
> 安装原则：程序放安装根目录，所有运行数据放安装根目录的 data 文件夹；不写 C 盘 AMS 数据目录，不依赖永久环境变量。  
> 编写日期：2026-10-24

---

## 目录

1. 一页安装清单
2. AMS Studio 是什么
3. 安装前准备
4. 从 GitHub 安装
5. 首次启动与初始化
6. 目录结构与文件位置
7. 启动器全部功能
8. 音乐插件页面与全部页签
9. 五个功能页的详细用法
10. 任务、进度、错误与产物
11. 高级用法：API、命令行、DSH 工具
12. 更新、修复、备份、迁移、卸载
13. 常见故障排查
14. 当前版本的能力边界
15. 安全、许可与使用建议
16. 附录：公开链接、校验值、快速检查表
17. 2026-10-24 维护修复

---

## 1. 一页安装清单

按下面顺序做，通常 20–40 分钟（取决于网速）：

1. 从官方 Release 下载 AMS-Studio-0.2.0-installlocal-setup.exe。
2. 安装到短路径，例如 F:/AMS-Studio 或 D:/AMS-Studio；不要装进 Program Files。
3. 启动 AMS Studio Launcher，先看“环境检测”：WebView2 和 VC++ 必须正常。
4. 点“检查组件清单”，确认有 11 个组件，点“下载并安装全部”，等待 11/11 成功（约 10.7 GB 下载）。
5. 点“启动内核”，等待“运行中”；点“停止”；点“安装音乐插件到内核”；再点“启动内核”。
6. 点 Gateway 的“启动”，等待“运行中”。
7. 点“扫描音源”，确认“曼波”存在；如果没有，点“安装默认曼波”。
8. 点“打开内核”，浏览器打开 AMS Kernel；点左侧“AMS 音乐”。
9. 先点“ping 自检”，再在“TTS 语音合成”页合成一句话，确认音频产物出现。
10. 可选：在“连接设置”填 DeepSeek 的网址和 Key，保存后重启内核，使用 AI 对话功能。

安装完成后，所有 AMS 数据都在：

~~~text
<你选择的安装根目录>/data
~~~

例如：

~~~text
F:/AMS-Studio/data
~~~

---

## 2. AMS Studio 是什么

AMS Studio 是一个“AI 音乐工作站”式的本地应用，分成四层：

| 层 | 作用 | 你看到的东西 |
|---|---|---|
| AMS Studio Launcher | 安装、环境检测、组件下载、启停内核和 Gateway、音源管理、MSVC 转换入口 | 深色启动器窗口 |
| AMS Kernel（DSH 内核） | 本地网页工作站；AI 对话、插件宿主、任务桥接 | 浏览器里的 AMS Kernel 页面 |
| Music Bundle 插件 | 给内核增加“AMS 音乐”页面和本地任务 API | 左侧侧栏“AMS 音乐” |
| Gateway + 后端运行时 | 真正执行 TTS、RVC 换声、pymss 分离、MSVC 转换 | 后台 Python / exe 进程，产物写入 data/artifacts |

当前公开版本实际包含的功能：

| 功能 | 状态 | 说明 |
|---|---|---|
| TTS 语音合成 | 可用 | Piper 离线中文 TTS，默认 zh_CN-huayan-medium，支持 data/tts_voices 用户音源 |
| 实时变声（Stage B MVP） | 可用 | 独立 VC 服务；麦克风/系统输出 → RVC → VB-Cable；设备枚举、启动/停止、RVC/直通 |
| 歌曲换源（RVC） | 可用 | 输入已分离纯人声 + 目标音色 pth/index，输出纯人声 wav |
| 人声分离（pymss） | 可用 | 输出 vocals / drums / bass / other 等 stems |
| MSVC 工具 | 可用 | 把 .msvc 容器转换为 RVC 可用的 .pth + conversion.json；不生成 index |
| 音源库 | 可用 | 扫描、默认曼波、从本地目录/voicepack.zip 安装 |
| 任务队列 / SSE / 取消 | 可用 | 每个任务有 job id、进度、阶段、错误、产物 |
| 外部素材导入到 data | 可用 | 自动复制到 data/projects/imports，避免后端直接读写外部路径 |
| 连接设置 | 可用 | 保存 DeepSeek 兼容 API 的网址和 Key，供内核 AI 对话使用 |
| 音源炼制（训练） | 未实现 | 页面是占位页；Gateway 未注册 train.rvc，提交会报错 |
| 项目管理 / 数据集 / 混音 / 评测 | 未实现 | 设计文档里有方案，当前构建没有对应页面和 Runner |

> 重要：这不是一套“已经包含全部 10 个设计插件”的成品。当前可用的核心是 TTS、RVC、pymss、MSVC 和任务系统。请以第 14 章为准。

---

## 3. 安装前准备

### 3.1 硬件与系统

| 项目 | 最低 | 推荐 |
|---|---|---|
| 系统 | Windows 10 x64 / Windows 11 x64 | Windows 11 x64 |
| 内存 | 8 GB | 16 GB 以上 |
| 磁盘 | 50 GB 可用 | 80 GB 以上 |
| GPU | 无（CPU 可跑，但慢） | NVIDIA，显存 8 GB 以上 |
| 网络 | 能访问 GitHub 和 huggingface.co | 稳定宽带 |
| 权限 | 普通用户即可 | 不要把程序装进 Program Files |

说明：

- RVC 和 pymss 默认使用 cuda:0；没有 NVIDIA 显卡时，在界面里选 cpu，功能仍可用，只是速度慢。
- TTS 不依赖 GPU。
- 首次组件下载约 10.7 GB，解包后会占更多空间；建议至少留 50 GB。

### 3.2 必须的 Windows 组件

启动器会在“环境检测”里检查：

| 检查项 | 说明 | 处理 |
|---|---|---|
| Windows / CPU / 内存 | 系统信息 | 正常即可 |
| GPU | 检测 nvidia-smi；没有则显示 WMI 显卡名 | 无 N 卡时后端用 CPU |
| WebView2（系统组件） | Launcher 的内嵌网页运行时 | 缺失则安装 Microsoft Edge WebView2 Runtime |
| VC++ 运行库 | vcruntime140.dll 是否存在 | 缺失则安装 Microsoft Visual C++ 2015-2022 x64 Redistributable |
| 内核 Node（内置） | 组件安装后出现 | 安装 kernel 组件后正常 |
| Gateway Python（内置） | 组件安装后出现 | 安装 gateway 组件后正常 |
| RVC 运行时（受管） | data/runtime/rvc/0.1.0 | 安装 runtime.rvc 后正常 |
| FFmpeg（受管） | 可选；不是所有输入都需要 | 转换 MP3/M4A/视频时再处理 |
| 曼波音源（内置资产） | data/voices/曼波/曼波.pth | 组件安装后正常 |
| 系统 Python / Node / Git | 仅开发用，可选 | 不影响运行 |
| 磁盘可用 | 各盘剩余空间 | 建议 50 GB 以上 |
| AMS 数据目录 | <安装根>/data | 自动创建 |

WebView2 安装：搜索“Microsoft Edge WebView2 Runtime 官方下载”，安装 Evergreen Standalone Installer（x64）。

VC++ 安装：搜索“Visual C++ Redistributable 2015-2022 x64 官方下载”，安装 vc_redist.x64.exe。

当前 feed 还包含一个 tool.vcrun 组件（约 0.5 MB），会把 VC++ 运行库放到 data/tools/vcrun，由启动器为 Gateway 子进程注入 PATH。它可以提高空系统下的可用性，但环境检测仍以系统 vcruntime 为准；看到“VC++ 缺失”时，仍建议装一遍系统 VC++。

### 3.3 安装路径建议

| 推荐 | 不推荐 |
|---|---|
| F:/AMS-Studio | C:/Program Files/AMS Studio |
| D:/AMS-Studio | 带大量空格或很长中文的路径 |
| 短路径、纯英文 | 网络盘、移动硬盘拔插中使用 |

原因：

- 安装器是每用户安装，不需要管理员；装到 Program Files 会引入权限问题。
- RVC / pymss 的 Python 路径很深，路径过长可能触发 Windows 260 字符限制。
- 所有数据会放在 <安装根>/data；迁移时整个根目录一起移动最稳。

---

## 4. 从 GitHub 安装

### 4.1 下载安装包

官方 Release 页面：

- https://github.com/yan-mc/ams-release/releases/tag/v0.2.1

安装包直链：

- https://github.com/yan-mc/ams-release/releases/download/v0.2.1/AMS-Studio-0.2.0-installlocal-setup.exe

当前文件信息：

| 项 | 值 |
|---|---|
| 文件名 | AMS-Studio-0.2.0-installlocal-setup.exe |
| 大小 | 2,438,343 字节（约 2.32 MiB） |
| SHA256 | D2DEFF5125CD857F41D9CE7A7C689E79A1DDA88AADA55F486DF4AE6060338B45 |
| 版本 | 0.2.0 |
| 签名 | 未做商业代码签名，Windows SmartScreen 可能提示未知发布者 |

只从上面的官方仓库下载。第三方转发的安装包不要用。

可选校验（PowerShell）：

~~~powershell
Get-FileHash .\AMS-Studio-0.2.0-installlocal-setup.exe -Algorithm SHA256
~~~

### 4.2 运行安装程序

1. 双击安装包。
2. 如果 SmartScreen 提示“Windows 已保护你的电脑”，点“更多信息” → “仍要运行”。
3. 选择安装目录，例如 F:/AMS-Studio。
4. 安装器不需要管理员权限；不要勾选装到 Program Files。
5. 等待完成。安装器可能会自动启动 AMS Studio Launcher；如果没有，去安装目录双击 ams-launcher.exe。

安装完成后根目录应该至少有：

~~~text
<安装根>/
  ams-launcher.exe
  uninstall.exe
  data/
~~~

### 4.3 安装后先确认“数据不落 C 盘”

打开 PowerShell，执行：

~~~powershell
Test-Path "$env:LOCALAPPDATA\com.yanmc.ams"
Test-Path "$env:LOCALAPPDATA\AMS Studio"
~~~

两个都应为 False。如果为 True，说明是旧版本残留或安装异常；先退出 AMS，删除这两个旧目录，再重新启动验证。

---

## 5. 首次启动与初始化

### 5.1 打开启动器

启动后你会看到深色页面，主要卡片：

- 连接设置
- 运行环境路径
- 环境检测
- 内核（DSH）
- Gateway
- 维护
- MSVC → pth 转换器
- 更新中心
- 组件安装
- 音源库
- 组件状态

新安装时，“组件安装”和“音源库”是空的，属于正常。

### 5.2 环境检测

先看“环境检测”：

- WebView2：如果是“未安装”，先装 WebView2。
- VC++ 运行库：如果显示“缺失”，先装 VC++ x64。
- 其他“未安装 / 未配置”在组件安装前是正常的。

### 5.3 安装全部组件

1. 找到“组件安装（本地 feed / GitHub Releases）”。
2. 点“检查组件清单”。
3. 正常会看到 11 个组件：

| 组件 | 类型 | 大小（约） | 来源 |
|---|---|---:|---|
| kernel.ams.dsh | kernel | 370.5 MB | GitHub |
| catalog.models | catalog | <1 MB | GitHub |
| asset.voicepack.manbo | asset | 130.4 MB | GitHub |
| service.gateway | service | 16.2 MB | GitHub |
| plugin.ams.music.bundle | plugin | <1 MB | GitHub |
| runtime.rvc | runtime | 6,871.2 MB | Hugging Face |
| runtime.pymss | runtime | 3,701.6 MB | Hugging Face |
| runtime.tts | runtime | 99.8 MB | Hugging Face |
| tool.msvc | tool | <1 MB | Hugging Face |
| runtime.pymss.python | runtime | 285.1 MB | Hugging Face |
| tool.vcrun | tool | 0.5 MB | Hugging Face |

4. 点“下载并安装全部”。
5. 保持启动器窗口打开。下载和安装期间不要手动删除 data/components/downloads。
6. 等待结果：11/11 成功。
7. 如果网络中断：重新点“下载并安装全部”，已下载的部分会复用；如果失败组件一直失败，先检查 GitHub / huggingface.co 是否可访问。

安装成功后：

- 内核位于 data/kernel/AMS Kernel.exe
- Gateway 位于 data/components/service.gateway/ams-gateway.exe
- RVC 位于 data/runtime/rvc/0.1.0
- pymss 位于 data/runtime/pymss/0.1.0
- TTS 位于 data/runtime/tts/0.1.0
- 曼波在 data/voices/曼波/
- 下载缓存留在 data/components/downloads/（可选清理）

### 5.4 初始化内核 profile 并安装音乐插件

插件需要内核 profile 先存在，所以顺序不能反：

1. 点“启动内核”。
2. 等待“状态：运行中”。
3. 点“停止”。
4. 点“安装音乐插件到内核”。
5. 看到“音乐插件已安装到 …/profiles/desktop；请重启 AMS Kernel”。
6. 再点“启动内核”，等待运行中。

如果第 4 步提示“内核 profile 尚未初始化”，说明第 1–3 步没有完成；重新执行一次。

### 5.5 启动 Gateway

1. 点 Gateway 卡片的“刷新”。
2. 点“启动”。
3. 等待“状态：运行中”，并显示端口、版本、任务数。
4. 如果 Gateway 显示“已安装未运行”，再点一次启动；仍失败则看第 13 章。

音乐功能依赖 Gateway。内核可以先开，Gateway 不开时页面能打开，但任务会报 Gateway 不可用。

### 5.6 安装 / 扫描音源

组件安装时 asset.voicepack.manbo 会自动把曼波安装到：

~~~text
data/voices/曼波/曼波.pth
data/voices/曼波/曼波.index
~~~

在“音源库”卡片：

1. 点“扫描音源”。
2. 应看到一个“曼波”，状态 ok，显示采样率、大小、index 有无。
3. 如果没有：点“安装默认曼波”。
4. 也可以用“从本地路径安装”：填一个包含 <名称>.pth 和可选 <名称>.index 的目录，或 voicepack.zip 路径，再点按钮。

音源目录结构必须类似：

~~~text
data/voices/
  曼波/
    曼波.pth
    曼波.index
  我的音色/
    我的音色.pth
    我的音色.index
~~~

### 5.7 打开内核并进入音乐页

1. 点“打开内核”；浏览器会打开形如 http://127.0.0.1:<端口>/?token=... 的地址。
2. 页面标题应为 AMS Kernel。
3. 左侧侧栏找到“AMS 音乐”（♪ 图标）。
4. 点击后进入“AMS 音乐工作站”。

如果侧栏没有“AMS 音乐”：

- 确认“安装音乐插件到内核”成功；
- 重启内核；
- 刷新浏览器页面；
- 仍然没有时看第 13 章。

### 5.8 可选：配置 AI 连接

连接设置只影响内核的 AI 对话，不影响 RVC / pymss / TTS / MSVC 这些本地任务。

在启动器“连接设置”或插件页“连接设置”中填：

| 字段 | 示例 |
|---|---|
| 网址 | https://api.deepseek.com/anthropic |
| Key | 你的 API Key |

点“保存连接”，然后重启内核生效。

保存位置：

~~~text
data/config/connection.json
data/kernel/dsh/.credentials.yaml
data/kernel/dsh/profiles/desktop/cordis.patch.yml
~~~

不会写入系统环境变量。

---

## 6. 目录结构与文件位置

### 6.1 安装根目录

~~~text
F:/AMS-Studio/                 ← 你选择的安装根
  ams-launcher.exe             ← 启动器
  uninstall.exe                ← NSIS 卸载器
  data/                        ← 所有 AMS 运行数据
    kernel/                    ← 内核程序与 DSH 数据
      AMS Kernel.exe
      dsh/
    components/                ← 组件程序
      service.gateway/
      plugin.ams.music.bundle/
      downloads/               ← 安装包缓存
    runtime/                   ← 后端运行时
      rvc/0.1.0/
      pymss/0.1.0/
      tts/0.1.0/
      appdirs/                 ← WebView2/应用数据
      ports.json
      gateway.token
      kernel.json
      kernel.log
    tools/                     ← MSVC / VC++ 运行库
      msvc2pth/0.1.0/
      vcrun/0.1.0/
    voices/                    ← 音源库
      曼波/
      我的音色/
    projects/                  ← 工程与导入素材
      imports/
    artifacts/                 ← Gateway 任务产物
    config/                    ← connection.json / runtime.json
    logs/                      ← 启动器/Gateway 日志
    tmp/                       ← 临时目录
~~~

### 6.2 重要文件说明

| 路径 | 用途 |
|---|---|
| data/kernel/AMS Kernel.exe | 内核程序 |
| data/kernel/dsh | DSH_HOME（会话、profile、凭据） |
| data/kernel/dsh/profiles/desktop | 插件安装的 profile |
| data/components/service.gateway | Gateway 程序 |
| data/runtime/ports.json | Gateway / 内核端口信息 |
| data/runtime/gateway.token | 本机 Gateway 访问 token，不要外传 |
| data/runtime/kernel.log | 内核日志 |
| data/config/runtime.json | RVC / pymss / MSVC 路径映射 |
| data/config/connection.json | AI 连接设置 |
| data/logs/launcher-cli.log | 命令行组件安装日志 |
| data/logs/launcher-cli-trace.log | 组件安装阶段追踪 |
| data/logs/gateway/ | Gateway 运行日志 |
| data/projects/imports | 外部素材复制后的位置 |
| data/artifacts | 任务产物 |

### 6.3 数据与程序分离

- 程序文件：安装根、data/kernel、data/components、data/runtime、data/tools。
- 用户资产：data/voices、data/projects、data/artifacts、data/config。
- 卸载器默认保留 data，方便重装/回滚；要彻底清理必须手动删整个安装根（见第 12 章）。

---

## 7. 启动器全部功能

启动器标题为 AMS Studio Launcher，版本 0.2.0。下面按卡片说明。

### 7.1 连接设置

用于给内核 AI 对话配置 API：

- 网址：DeepSeek 兼容地址，默认示例 https://api.deepseek.com/anthropic
- Key：API Key；已保存后输入框留空表示不修改
- “保存连接”：写入 data，不写环境变量
- “刷新”：读取当前是否已配置

保存后要重启内核才生效。

### 7.2 运行环境路径

显示并允许修改：

- RVC Python
- RVC 根目录
- pymss Python
- pymss 根目录
- MSVC 工具（msvc2pth.py）

组件安装会自动填好。除非你明确知道路径含义，不要手动改。保存后 Gateway 需要重启。

### 7.3 环境检测

显示操作系统、CPU、内存、GPU、WebView2、VC++、内置 Node、Gateway Python、RVC、FFmpeg、曼波、系统 Python/Node/Git、磁盘、AMS 数据目录。

- “重新检测”：刷新。
- 红色/黄色项不一定阻断；但 WebView2、VC++、内核、Gateway、RVC、曼波是关键项。

### 7.4 内核（DSH）

| 按钮 | 作用 |
|---|---|
| 启动内核 | 启动 data/kernel/AMS Kernel.exe，写 kernel.json / kernel.log |
| 停止 | 停止内核进程树 |
| 打开内核 | 用默认浏览器打开带 token 的内核 URL |
| 刷新 | 更新状态、端口、PID、DSH_HOME |

内核启动需要先安装 kernel 组件。首次启动会创建 profile。

### 7.5 Gateway

| 按钮 | 作用 |
|---|---|
| 启动 | 启动 data/components/service.gateway/ams-gateway.exe |
| 停止 | 停止 Gateway |
| 刷新 | 更新健康状态、端口、版本、任务数、数据目录 |

Gateway 是任务的真正执行入口；不启动它，插件页无法提交任务。

### 7.6 维护

- “卸载 AMS Studio…”：打开 NSIS 卸载器。
- 注意：卸载器默认保留 data，只删程序和快捷方式；不会自动删除你的音源、工程、产物。
- 彻底清理见第 12 章。

### 7.7 MSVC → pth 转换器

启动器自带的快捷转换入口，适合只想转一个 .msvc 文件的情况：

| 字段 | 说明 |
|---|---|
| 输入 .msvc | 要转换的 MSVC 容器文件 |
| 参考 pth | 可选，同一架构的 RVC pth，用于参考校验 |
| 输出目录 | 留空时自动使用 data/artifacts |
| 音源名 | 可选；不能包含路径分隔符 |

点“选择”可以打开文件对话框；点“提交转换”；下面显示任务状态、进度、阶段。产物：

~~~text
<输出目录>/<音源名>/<音源名>.pth
<输出目录>/<音源名>/conversion.json
~~~

### 7.8 更新中心

- 选择清单 JSON → 读取清单 → 验证签名。
- 显示 schema/channel/sequence/signature/组件列表。
- 这是手动验证清单的入口；不是全自动后台更新。
- 公开 feed 在 Release 的 ams-components.json，带 Ed25519 签名和 sequence 防回滚。

### 7.9 组件安装

| 按钮 | 作用 |
|---|---|
| 检查组件清单 | 从公开 feed 读取 11 个组件 |
| 下载并安装全部 | 下载 + 校验 sha256 + 解包到 data |
| 安装音乐插件到内核 | 把 3 个插件 tgz 安装到内核 profile；需要先停止内核 |

使用顺序：检查清单 → 安装全部 → 启动内核一次 → 停止 → 安装插件 → 再启动内核。

### 7.10 音源库

| 元素 | 说明 |
|---|---|
| 扫描音源 | 扫描 data/voices，调用 Gateway /voices |
| 安装默认曼波 | 从内置/组件资产安装曼波到 data/voices/曼波 |
| 从本地路径安装 | 填本地曼波目录或 voicepack.zip，复制到 data/voices/曼波 |

音源必须满足 voices/<名称>/<名称>.pth（index 可选但推荐）。不是标准目录结构的文件不会被正确识别。

### 7.11 组件状态

显示 kernel、gateway、plugin、runtime、音源等组件是否齐全。用于快速判断“为什么某个按钮是灰的”。

### 7.12 使用说明 / 关于 AMS

启动器右上角有 **使用说明 / 关于 AMS** 按钮，点击后在启动器内部打开说明页（不是外部浏览器）。内容包括：整体结构、首次使用顺序、当前可用功能、data 目录、文件选择/打开目录、当前未实现功能、上游项目和卸载数据说明。点“关闭”或遮罩返回启动器。

---

## 8. 音乐插件页面与全部页签

### 8.1 页面入口

启动器 → 打开内核 → 浏览器 AMS Kernel → 左侧“AMS 音乐”（♪）。

页面标题：AMS 音乐工作站。

顶部和说明区会显示：

- Host 插件 @ams/plugin-core
- 页面版本
- 数据经 Host 插件鉴权桥接
- 任务进度走 SSE，断线自动重连/降级轮询

### 8.2 页面结构

从上到下：

1. 连接设置（AI 网址 / Key）
2. 功能页签：实时变声 / 音源炼制（占位）/ 人声分离 / 歌曲换源 / TTS 语音合成
3. 任务（Gateway Job）：任务表、ping 自检、刷新列表
4. 运行状态：Host 插件、已注册工具、Gateway、任务类型、页面鉴权

### 8.3 五个页签的总表

| 页签 | 任务类型 | 当前状态 | 输入 | 输出 |
|---|---|---|---|---|
| 实时变声 | vc service | Stage B MVP | 麦克风/系统输出、RVC 音源、VB-Cable 输出 | 实时音频到 VB-Cable |
| 音源炼制 | train.rvc | 未注册，占位 | 数据集目录、模型名、epochs、batch、采样率 | 无 |
| 人声分离 | separate.pymss | 可用 | 整曲、模型、设备 | stems wav + manifest |
| 歌曲换源 | convert.rvc | 可用 | 已分离纯人声、目标音色、变调、index、设备 | 纯人声 wav + json |
| TTS 语音合成 | tts.synthesize | 可用 | 文本、模型 | tts.wav |

### 8.4 所有页签共用的“选择文件 / 导入到 data”

带文件路径的字段下方现在有两个按钮：

- **选择文件**：打开系统文件选择窗口；选中后浏览器把文件上传给 Host 插件，Host 复制到 `data/projects/imports/...`，表单自动回填 data 内路径。
- **导入到 data**：如果文本框里已经有外部绝对路径，可以用这个按钮让 Gateway 复制到 data。

上传后路径形如：

~~~text
data/projects/imports/unscoped/<随机id>/<文件名>
~~~

如果按钮或目录打开失败，页面会显示失败原因；音源/产物目录按钮旁也会显示固定相对路径，可直接按路径用资源管理器打开。

这样做是为了满足“后端只读 data 内素材”的边界。即使你不点，Gateway 在提交任务时也会自动复制外部输入；但先选择文件更直观。

### 8.5 任务区

任务区右上角有 **打开产物目录** 按钮；旁边显示 `产物路径：data/artifacts/{job_id}`，按钮打不开时可直接按这个路径找。

| 列 | 含义 |
|---|---|
| Job ID | 任务 id，产物目录名就是它 |
| 类型 | 如 tts.synthesize / convert.rvc |
| 状态 | queued / running / stopping / succeeded / failed / cancelled |
| 进度 | 百分比 |
| 阶段 | 如 tts / save / done |
| 操作 | 运行中/排队中可点“取消” |

按钮：

- “ping 自检”：提交一个 ping 任务，验证插件→Gateway 通路。
- “刷新列表”：重新读取最近任务。
- 单个任务的“取消”：排队中立即取消；运行中先 stopping，然后 Gateway 杀进程树。

### 8.6 运行状态

显示：

- Host 插件及版本
- 已注册工具（music_ping / music_job_submit / music_job_status / music_job_list / music_job_cancel）
- Gateway ok/down 和 base URL
- Gateway 任务类型及是否 GPU
- 页面鉴权方式（session；Gateway token 不出 Host）

---

## 9. 五个功能页的详细用法

### 9.1 音源炼制（当前不可用）

页面字段：

- 训练数据集目录
- 模型名
- Epochs（默认 100）
- Batch size（默认 8）
- 采样率（默认 40000）
- 开始炼制

当前版本不要用这个页签做训练。原因：

- Gateway 注册的任务类型里没有 train.rvc；
- 页面会显示“Gateway 当前未注册 train.rvc”，提交会失败；
- 训练 Runner、数据集规范、模型导出在当前构建中未实现。

如果你需要训练，请使用外部 RVC WebUI，或等后续版本。这个页签仅作为未来功能占位保留。

### 9.2 人声分离（pymss）

用途：把歌曲拆成人声、鼓、贝斯、其他等 stems。

字段：

| 字段 | 必填 | 说明 |
|---|---|---|
| 输入音频 | 是 | wav/flac/mp3/m4a 等；建议先转 WAV |
| 分离模型 | 是 | scnet_checkpoint_musdb18 或 bs_roformer_voc_hyperacev2 |
| 设备 | 否 | auto / cuda:0 / cpu；默认 auto |

操作：

1. 点输入框旁“导入到 data”（或直接填 data 内路径）。
2. 选择模型。
3. 选择设备；无 N 卡选 cpu。
4. 点“开始分离”。
5. 在任务表看状态；成功后去产物目录。

产物：

~~~text
data/artifacts/{job_id}/stems/<输入文件名>_{stem}.wav
data/artifacts/{job_id}/stems/separation-manifest.json
~~~

stem 名称由模型决定，常见为 vocals / drums / bass / other。具体以 manifest 为准。

提示：

- 首次使用某个 pymss 模型时，如果模型文件不在 data/runtime/pymss/0.1.0/models，会失败；把模型放到该目录，或按 pymss 官方方式下载。
- GPU 任务串行执行；同时提交多个会排队。
- 分离出来的人声可以直接填入“歌曲换源”；不会自动混伴奏。
- 没有 GPU 时选 cpu，速度明显变慢。

### 9.3 歌曲换源（RVC 纯人声）

用途：把已分离的纯人声换成目标音色；输出纯人声，不与伴奏混合。

字段：

| 字段 | 必填 | 默认 | 说明 |
|---|---|---|---|
| 纯人声输入 | 是 | — | 已分离好的 wav/flac/mp3/m4a 人声 |
| 目标音色 | 是 | 自动选第一个已扫描音源 | 从 data/voices 动态加载；也可传 pth 路径 |
| 变调（半音） | 否 | 0 | -24..24；男声→女声常用 +12，女声→男声常用 -12 |
| 索引混合率 | 否 | 0.75 | 0..1；越高越贴近目标音色，过高可能失真 |
| 设备 | 否 | cuda:0 | cuda:0 / cuda / cpu；无 N 卡选 cpu |

操作：

1. 导入已分离纯人声。
2. 从“目标音色”下拉选择音源；下拉会默认选第一个，页面会做必填校验。
3. 设置变调与索引混合率。
4. 选择设备。
5. 点“开始换源”。
6. 成功后去 data/artifacts/{job_id}/ 找输出。

产物：

~~~text
data/artifacts/{job_id}/<输入文件名>_<音色名>.wav
data/artifacts/{job_id}/<输入文件名>_<音色名>.json
~~~

提示：

- 输出 WAV 是 24-bit PCM。
- index 能提升相似度，但如果音色本身质量差，降低 index 可能更自然。
- RVC 模型有 32k/40k/48k 等采样率；不匹配可能报错或音质异常，优先用同一来源的模型和 index。
- 音色目录名尽量用英文/数字/短中文；如果命令行调用遇到编码问题，优先用界面。
- 没有 GPU 时选 cpu 可用，但很慢。

### 9.4 MSVC 工具（独立转换，不是歌曲换源）

用途：把心月 MSVC 容器转换为标准 RVC .pth。它只做模型格式转换，不参与歌曲换源，也不生成 index；index 可以由用户另外提供，缺失时用 index_rate=0。

字段：

| 字段 | 必填 | 说明 |
|---|---|---|
| 输入 .msvc 文件 | 是 | MSVC 容器，不是 .pth，也不是音频 |
| 模型名（可选） | 否 | 输出音源名；不能包含路径分隔符 |
| 参考 .pth（可选） | 否 | 同架构 RVC pth，用于参考校验 |
| 输出目录（data 内） | 当前版本建议必填 | 例如 data/artifacts/msvc 的完整路径 |

当前 MSVC 工具只在启动器的“MSVC → pth 转换器”卡片里使用，不在音乐插件页签中。Gateway 的 MSVC Runner 要求 out_dir 非空；启动器卡片在留空时会自动填 data/artifacts。

操作：

1. 选择 .msvc 文件。
2. 可选选择参考 .pth。
3. 填输出目录，例如 F:/AMS-Studio/data/artifacts/msvc。
4. 填音源名（可选）。
5. 点“提交转换”。
6. 任务成功后查输出目录。

产物：

~~~text
<输出目录>/<音源名>/<音源名>.pth
<输出目录>/<音源名>/conversion.json
~~~

提示：

- 输出 .pth 可以放回 data/voices/<名称>/<名称>.pth，再扫描音源，用于 RVC。
- 该任务 CPU 即可，不需要 GPU。
- 转换报告记录 tensor 数、warnings、工具版本，出现 warning 时先看 conversion.json。
- 如果 .msvc 损坏或 reference 架构不匹配，会失败；看任务错误和 stderr。

### 9.5 TTS 语音合成

用途：用 Piper 把文字合成中文语音。

字段：

| 字段 | 必填 | 说明 |
|---|---|---|
| 要合成的文本 | 是 | 最多 2000 字 |
| TTS 模型 | 否 | 先扫描 data/tts_voices，再扫描 runtime；**同名模型只显示一次，用户目录版本优先** |

操作：

1. 输入文本。
2. 选择模型（可留空使用默认）。
3. 点“开始合成”。
4. 任务成功后找到 wav。

产物：

~~~text
data/artifacts/{job_id}/tts.wav
~~~

提示：

- 中文效果取决于模型；默认 huayan 是中文女声。
- 标点会影响停顿；不要一次塞太长的无标点文本。
- 单次最多 2000 字；超时会报错。
- 如果报“TTS 运行时缺少 piper.exe 或模型文件”，重装 runtime.tts。
- 如果 Piper 启动失败且系统缺少 VC++，安装 VC++ x64，或确认 tool.vcrun 已安装。
- 该功能不需要 GPU，不需要 DeepSeek Key。

### 9.6 实时变声（Stage B MVP）

用途：把麦克风或电脑系统输出实时换成目标音色，送到 VB-Cable，供 OBS、腾讯会议等选择为麦克风。

前提：

- 本机已安装 VB-Cable；AMS 能检测到 `VSpk`/`CABLE` 输出。
- 系统输出优先使用随 `service.vc` 内置的 WASAPI Loopback（data/components/vc-service/vendor），无需 pip、无需系统 Python 包；内置不可用时仍可回退到 Windows“立体声混音”。
- RVC 音源放在 `data/voices/<名称>/<名称>.pth`；index 可选，缺失时 index_rate=0。页面会自动匹配同名 index；index 不兼容时服务会禁用 index 并在状态里提示，不会让实时流启动失败。

操作：

1. 打开“实时变声”页签；首次打开会自动拉起独立 VC 服务并枚举设备。
2. 输入设备只保留：默认麦克风、另一个物理麦克风、内置 `WASAPI Loopback · <扬声器>`；不再列出 MME/DirectSound/WASAPI/WDM-KS 的重复项。
   如需麦克风和电脑声音同时进入变声器，把“叠加系统声音（可选）”设为对应 WASAPI Loopback。
3. 输出设备只保留两个主选项：**VB-Cable 虚拟输出（给 OBS/会议）** 和 **默认输出（当前耳机/扬声器）**。
   主输出选 VB-Cable；下面的“监听输出（可选，耳机）”选默认输出，就能同时给 OBS 和耳机。
   已隐藏 Microsoft 声音映射器、主声音驱动程序、重复的 Realtek 设备名和截断的 MME 名称。
4. 模式选择 `RVC 变声`；选择目标音色、变调、索引混合率、F0 方法、延迟模式（低延迟 / 稳定）。
5. 点“启动实时变声”；状态显示运行中、采样率、frames，以及实时推理耗时和估算延迟。
6. 在 OBS/腾讯会议里，把麦克风选成 `CABLE-A Output`（VB-Audio CABLE-A）。
7. 运行中改选另一个音色后点“切换音色”；同一采样率会直接替换推理引擎，不重启音频流；采样率变化时才短暂重启。
8. 点“停止”释放设备。

当前 MVP 限制：

- “切换音色”已支持同采样率无缝替换；不同采样率模型会短暂重启流。
- 延迟模式增加“极低延迟（实验）”：block 50 ms + buffer 150 ms，实测估算约 284 ms（infer p95 约 47 ms）；仍是软件估算，不等同外部仪器 p95。
- 页面显示输入/输出电平和推理 p95，便于判断断流、静音和性能问题。
- 系统输出优先走内置 WASAPI Loopback；该组件固定在 data 内，不写用户目录、不执行 pip。若内置组件缺失，才回退到立体声混音。
- 内置 loopback 的二进制是 PyAudioWPatch 0.2.12.9（Apache-2.0，自带 PortAudio DLL）；AMS 不修改系统音频设备。
- VB-Cable 仍是外部虚拟声卡驱动，AMS 不自动安装驱动；新系统需要先安装 VB-Cable。
- 不包含自研虚拟声卡驱动；使用 VB-Cable。
- 延迟由 block_time 0.25s + RVC extra buffer 决定；本版本未做延迟验收。

### 9.7 连接设置

见 5.8。补充：

- 连接设置只对内核 AI 对话生效；本地音乐任务不需要。
- Key 保存在 data，不要分享整个 data 目录。
- 保存后必须重启内核。
- 如果 AI 页面仍要求 API Key，检查 data/config/connection.json 和 profile patch 是否存在，并重启内核。

---

## 10. 任务、进度、错误与产物

### 10.1 状态机

| 状态 | 含义 |
|---|---|
| queued | 排队 |
| running | 执行中 |
| stopping | 取消中，等待进程退出 |
| succeeded | 成功 |
| failed | 失败，error 字段有原因 |
| cancelled | 已取消 |

### 10.2 进度与阶段

常见阶段：

- tts / tts.synthesize：TTS 进度
- save / done：写出产物
- convert.starting / convert.calibrating / convert.done：MSVC
- RVC 的加载/推理/保存阶段
- pymss 的加载/分离/保存阶段

### 10.3 错误怎么读

任务失败时先看错误码和 message：

| 错误 / 现象 | 可能原因 | 处理 |
|---|---|---|
| INPUT_NOT_FOUND | 输入路径不存在 | 重新选文件，先“导入到 data” |
| 缺少参数 out_dir | MSVC 输出目录为空 | 填 data 内输出目录 |
| Gateway 当前未注册 train.rvc | 训练功能未实现 | 不要用音源炼制页签 |
| BACKEND_NOT_CONFIGURED | TTS 运行时未配置 | 安装 runtime.tts 并重启 Gateway |
| TTS_FAILED | Piper 失败 | 看 VC++、模型、文本长度 |
| RVC_CONVERT_FAILED | 音色/模型/GPU 问题 | 看 stderr 和 voice 路径 |
| PYMSS_SEPARATE_FAILED | 模型/GPU/环境 | 看 stderr；无 GPU 选 cpu |
| 422 未知任务类型 | 后端未注册 | 检查 Gateway 版本 |
| Gateway 不可用 | Gateway 没启动 | 启动 Gateway |

### 10.4 产物在哪

所有 Gateway 任务产物在：

~~~text
data/artifacts/{job_id}/
~~~

当前插件页没有下载按钮；请在 Windows 资源管理器打开该目录复制文件。这是当前版本的 UI 限制。

Msvc 的产物可能在用户指定的 out_dir，默认启动器卡片会用 data/artifacts。

---

## 11. 高级用法：API、命令行、DSH 工具

> 普通用户可以不看这一章。高级操作仍然遵守“所有输入输出在 data 内”的边界。

### 11.1 插件桥 API

页面通过本机 Host 插件访问 Gateway：

~~~text
GET  /ams/api/session       获取浏览器会话 Cookie（同源 loopback）
GET  /ams/api/summary       插件 + Gateway 摘要
GET  /ams/api/voices        音源列表
GET  /ams/api/tts/models    TTS 模型列表
POST /ams/api/jobs          提交任务 {type, params}
GET  /ams/api/jobs          任务列表
GET  /ams/api/jobs/{id}     任务详情
POST /ams/api/jobs/{id}/cancel
GET  /ams/api/jobs/{id}/events   SSE
POST /ams/api/projects/import   {source_path, project_id?}
GET  /ams/api/connection / POST /ams/api/connection
~~~

浏览器会自动带 session cookie；机器调用可以带 Gateway Bearer token，但不要外传。

### 11.2 Gateway API

Gateway 只监听 127.0.0.1，端口和 token 在：

~~~text
data/runtime/ports.json
data/runtime/gateway.token
~~~

基础路径：http://127.0.0.1:{port}/api/v1

常用端点：

| 方法 | 路径 | 用途 |
|---|---|---|
| GET | /health | 健康检查、任务类型 |
| POST | /jobs | 创建任务 |
| GET | /jobs | 列出任务 |
| GET | /jobs/{id} | 任务详情 |
| POST | /jobs/{id}/cancel | 取消 |
| GET | /jobs/{id}/events | SSE |
| GET | /jobs/{id}/artifacts | 产物列表 |
| GET | /artifacts/{artifact_id}/download | 下载产物 |
| GET | /voices | 音源 |
| POST | /voices/scan | 重新扫描 |
| GET | /tts/models | TTS 模型 |
| POST | /projects/import | 导入素材 |

请求需要：

~~~text
Authorization: Bearer {gateway.token}
~~~

### 11.3 命令行

启动器支持：

~~~text
ams-launcher.exe --install-components
ams-launcher.exe --start-kernel
~~~

- --install-components：静默安装全部组件，日志在 data/logs/launcher-cli.log。
- --start-kernel：启动内核后退出。
- 不建议普通用户使用内部清理参数。

### 11.4 DSH 工具

插件在 Host 侧注册了 5 个模型可调用工具：

- music_ping
- music_job_submit
- music_job_status
- music_job_list
- music_job_cancel

如果配置了 DeepSeek Key，可以在 AI 对话里让它提交/查询任务。注意：

- 工具参数和 UI 不完全相同（例如 RVC 工具用 voice，而 UI 用 voice_name）。
- 工具不支持 tts.synthesize；TTS 请用 UI。
- 工具只负责提交，产物仍需去 data/artifacts 取。
- 普通用户建议用 UI，避免参数不一致。

---

## 12. 更新、修复、备份、迁移、卸载

### 12.1 更新组件

1. 打开启动器。
2. 组件安装 → 检查组件清单。
3. 如果有新版本，点“下载并安装全部”。
4. 安装后重启 Gateway 和内核。
5. 如果只更新了部分组件，至少重启相关服务。

更新使用签名 feed，sequence 递增防回滚；不要手工替换 data 里的文件来“升级”。

### 12.2 修复

- 组件损坏/缺失：重新“下载并安装全部”，会覆盖重建。
- 插件丢失：重新“安装音乐插件到内核”。
- 音源丢失：重新安装曼波/本地音源。
- Gateway 起不来：先重启启动器，再看日志；必要时重装 service.gateway。
- 修复不会删除 voices/projects/artifacts。

### 12.3 备份

建议在完全退出 AMS（内核、Gateway、启动器）后备份整个：

~~~text
<安装根>/data
~~~

至少备份：

~~~text
data/voices
data/projects
data/artifacts
data/config
data/kernel/dsh
~~~

data/components/downloads 是下载缓存，可以不备份。

### 12.4 迁移安装目录

最稳妥：卸载程序但保留 data → 重新安装到新目录 → 把旧 data 复制到新目录。  
不要只移动 exe 而不管注册表/快捷方式；启动器按 exe 所在目录推导 data，移动整个根目录通常可用，但建议重建快捷方式并重新验证环境检测。

### 12.5 卸载

1. 启动器 → 维护 → “卸载 AMS Studio…”。
2. 确认卸载；NSIS 卸载器会删除程序文件、快捷方式、HKCU 卸载项。
3. 默认保留 data，包括音源、工程、产物、运行时。
4. 如果要彻底清理：
   - 关闭所有 AMS 进程；
   - 确认路径是 <安装根>（例如 F:/AMS-Studio）；
   - 删除整个安装根目录（这会删除 data）；
   - 检查 C:/Users/<你>/AppData/Local/com.yanmc.ams 和 AppData/Roaming/com.yanmc.ams 不存在；
   - 不需要动其他应用的数据。

> 卸载器的设计决定是“默认保留用户数据”，不是 bug。要完全删除必须手动删根目录。删除前请确认没有其他程序放在同一目录。

### 12.6 重装

- 覆盖安装会复用 data。
- 想从零开始：卸载 → 删除整个安装根 → 重新安装 → 重新下载组件。
- 本次维护已验证：全新安装、组件下载、卸载、data 清理均可用。

---

## 13. 常见故障排查

### 13.1 安装器 / SmartScreen

| 现象 | 处理 |
|---|---|
| SmartScreen 未知发布者 | 官方包未商业签名；确认下载地址后“更多信息 → 仍要运行” |
| 安装被拦截 | 检查杀毒软件；不要把安装目录设为只读 |
| 安装后没有快捷方式 | 直接去安装根运行 ams-launcher.exe |
| 安装路径含中文/空格导致异常 | 换短英文路径重装 |

### 13.2 WebView2 / VC++

| 现象 | 处理 |
|---|---|
| 启动器空白/打不开 | 安装 WebView2 Evergreen x64 |
| TTS 报 Piper 失败 | 安装 VC++ 2015-2022 x64；确认 tool.vcrun 组件已安装 |
| RVC/pymss 报 DLL 缺失 | 安装 VC++ x64；确认 tool.vcrun 存在 |

### 13.3 组件下载

| 现象 | 处理 |
|---|---|
| 检查清单失败 | 确认能访问 GitHub Release；重试 |
| RVC/pymss 下载慢或失败 | 确认能访问 huggingface.co；重试；不要同时开多个安装 |
| 某个组件 sha256 失败 | 删除该组件在 data/components/downloads 下的缓存后重装 |
| 安装到一半关闭 | 重新打开，再点下载并安装全部；缓存可复用 |
| 磁盘不足 | 清理 downloads 或换盘；至少 50 GB |

### 13.4 内核

| 现象 | 处理 |
|---|---|
| 启动内核按钮灰 | 先安装 kernel 组件 |
| 内核起不来 | 看 data/runtime/kernel.log；确认 data/kernel/AMS Kernel.exe 存在 |
| 打开内核 401/白屏 | 用启动器“打开内核”带 token 的 URL；不要手输裸地址 |
| 侧栏没有 AMS 音乐 | 停止内核 → 安装音乐插件 → 启动内核；刷新页面 |
| 安装插件提示 profile 未初始化 | 先启动内核一次，再停止，再安装 |

### 13.5 Gateway

| 现象 | 处理 |
|---|---|
| Gateway 未运行 | 启动；看 data/logs/gateway；确认 service.gateway 已安装 |
| 端口冲突 | 关闭占用端口的程序，重启 Gateway；端口在 ports.json |
| 任务提交 401 | 通过插件页提交；不要手工裸调 Gateway 不带 token |
| 任务一直排队 | 检查是否有 GPU 任务在执行；取消不需要的任务 |

### 13.6 TTS

| 现象 | 处理 |
|---|---|
| 缺少 piper.exe/模型 | 重装 runtime.tts |
| 合成失败/超时 | 文本降到 2000 字以内；安装 VC++；重试 |
| 没有 TTS 模型下拉 | 确认 runtime.tts/models 有 .onnx；重启 Gateway |
| 输出找不到 | data/artifacts/{job_id}/tts.wav |

### 13.7 RVC

| 现象 | 处理 |
|---|---|
| 目标音色为空 | 启动 Gateway → 扫描音源；安装曼波 |
| 音源不存在 | 检查 data/voices/<名称>/<名称>.pth；重新扫描 |
| CUDA 不可用 | 设备选 cpu；或安装 NVIDIA 驱动 |
| 输出质量差 | 调整 pitch/index_rate；换匹配采样率模型 |
| 中文路径异常 | 优先用 UI；命令行调用时注意 UTF-8/代码页 |

### 13.8 pymss

| 现象 | 处理 |
|---|---|
| 模型不存在 | 把模型放到 data/runtime/pymss/0.1.0/models |
| CUDA 不可用 | 设备选 cpu |
| 分离很慢 | GPU 串行；大文件耐心等待 |
| Python 启动失败 | 确认 runtime.pymss.python 组件已安装；重装 pymss |

### 13.9 MSVC

| 现象 | 处理 |
|---|---|
| 缺少 out_dir | 填 data 内输出目录；或用启动器 MSVC 卡片 |
| 输入不是 .msvc | 确认文件扩展名和格式；.pth 不能当输入 |
| reference 不匹配 | 去掉参考 pth 再试，或换同架构参考 |
| 转换报告 warning | 打开 conversion.json 查看；必要时换工具版本 |

### 13.10 数据 / C 盘

| 现象 | 处理 |
|---|---|
| C 盘出现 com.yanmc.ams | 旧版残留；退出 AMS 后删除该目录；新版数据应在安装根/data |
| 想确认数据位置 | 启动器环境检测“AMS 数据目录” |
| 磁盘不够 | 清 data/components/downloads；备份后清 artifacts/projects |
| 重装后数据还在 | 正常，卸载器保留 data；要清空需手动删根目录 |

---

## 14. 当前版本的能力边界

### 14.1 已实现

- 启动器安装/环境检测/组件下载/内核/Gateway/音源/MSVC/更新验签/卸载
- Gateway 任务：ping、mock.long（测试）、convert.msvc、convert.rvc、separate.pymss、tts.synthesize
- 插件页面：任务表、SSE、取消、ping、连接设置、导入到 data
- 4 个功能页签（音源炼制仍是占位；歌曲换源已改为 RVC 纯人声）
- 数据全部在 data；无持久环境变量；C 盘无 AMS 数据目录

### 14.2 未实现 / 不要期待

| 功能 | 状态 |
|---|---|
| RVC 音源训练（train.rvc） | 未实现；页签占位 |
| 数据集工作室 | 未实现 |
| 音色库高级管理/试听 | 未实现（只有扫描/安装/下拉） |
| 混音母带 | 未实现 |
| 评测器 | 未实现 |
| 拖拽式项目管理 | 未实现 |
| 任务产物下载按钮 | 未实现；用资源管理器 |
| 自动后台更新 | 未实现；手动组件安装/验签 |
| 实时变声 | Stage B MVP 已实现（RVC/直通、设备枚举、VB-Cable、切换音色） |
| 模型市场/一键下载 | 未实现 |

> 设计文档里的 10 个插件是路线图，不是当前安装包的实际功能。写说明书时按“当前可用”使用，避免把占位页当功能。

---

## 15. 安全、许可与使用建议

### 15.0 数据与临时目录边界

AMS 运行时的可写数据、临时文件、缓存都在安装根 `data` 下：

- `data/tmp`：TEMP/TMP/TMPDIR
- `data/cache/torch`、`data/cache/torch-extensions`：PyTorch
- `data/cache/huggingface`：HuggingFace/Transformers
- `data/cache/matplotlib`、`data/cache/numba`、`data/cache/pip`、`data/cache/cuda`
- `data/cache/pycache`：Python 字节码缓存
- `data/runtime`、`data/components`、`data/artifacts`、`data/voices`、`data/tts_voices`

启动器启动 Gateway/Kernel/VC 服务时会显式设置这些环境变量；VC 服务自身也会兜底设置。不使用 `%LOCALAPPDATA%`、`%APPDATA%` 或用户目录作为 AMS 运行数据目录。安装器自身的 NSIS 临时文件属于安装器行为，不在运行时数据范围内。

- 本机回环：Gateway 只监听 127.0.0.1，token 在 data/runtime；不要把 token 发人。
- API Key：保存在 data/config/connection.json 和 DSH 凭据；不要分享 data 目录。
- 音源/模型许可：曼波是第三方音色资产；公开分发/商用前确认授权。用户自备音源同样要确认许可。
- 生成内容：遵守当地法律和平台规则；不要用他人声音做未经授权的冒充。
- 未签名：安装包未商业签名，SmartScreen 可能警告；只从官方 GitHub 下载。
- 杀毒软件：RVC/pymss 会启动 Python 子进程，可能被误判；只对可信安装目录做必要排除，不要关闭全局防护。
- 备份：重要工程/音源定期备份 data；卸载前先备份。

---

## 16. 附录

### 16.1 公开链接

| 用途 | 链接 |
|---|---|
| Release 页面 | https://github.com/yan-mc/ams-release/releases/tag/v0.2.1 |
| 安装包 | https://github.com/yan-mc/ams-release/releases/download/v0.2.1/AMS-Studio-0.2.0-installlocal-setup.exe |
| 组件 feed | https://github.com/yan-mc/ams-release/releases/latest/download/ams-components.json |
| Hugging Face 数据集 | https://huggingface.co/datasets/yan2002/AI-MUSIC-STUDIO |
| 源码 launcher | https://github.com/yan-mc/ams-launcher |
| 源码 services | https://github.com/yan-mc/ams-services |
| 源码 plugins | https://github.com/yan-mc/ams-plugins |

当前 feed：sequence 2042，12 个组件。安装包大小 2,438,343 字节，SHA256 见 4.1。

### 16.2 安装后最小验收清单

- [ ] 启动器能打开，环境检测显示 WebView2、VC++ 正常
- [ ] 组件清单 11 个，安装结果 11/11 成功
- [ ] 内核能启动、停止，profile 存在
- [ ] 音乐插件安装成功，重启内核后侧栏出现“AMS 音乐”
- [ ] Gateway 运行中，ping 自检成功
- [ ] 音源库能看到曼波
- [ ] TTS 能生成 tts.wav
- [ ] RVC 能用曼波生成 wav（无 N 卡选 cpu）
- [ ] pymss 能生成 stems（无 N 卡选 cpu）
- [ ] MSVC 转换能生成 pth + conversion.json
- [ ] 产物在 data/artifacts，C 盘无 com.yanmc.ams
- [ ] 退出后无 AMS 进程；卸载器能正常打开

### 16.3 关键路径速查

| 内容 | 路径 |
|---|---|
| 全部数据 | <安装根>/data |
| 音源 | data/voices/<名称>/<名称>.pth |
| 任务产物 | data/artifacts/{job_id}/ |
| 外部素材导入 | data/projects/imports/ |
| TTS 模型 | data/tts_voices/*.onnx + data/runtime/tts/0.1.0/models/*.onnx |
| pymss 模型 | data/runtime/pymss/0.1.0/models/ |
| MSVC 工具 | data/tools/msvc2pth/0.1.0/msvc2pth.py |
| 运行环境配置 | data/config/runtime.json |
| AI 连接配置 | data/config/connection.json |
| 内核日志 | data/runtime/kernel.log |
| Gateway 日志 | data/logs/gateway/ |
| 组件安装日志 | data/logs/launcher-cli.log |

### 16.4 术语

| 术语 | 说明 |
|---|---|
| RVC | Retrieval-based Voice Conversion，检索式变声 |
| pymss | Python 音乐源分离工具集 |
| MSVC | 心月的模型容器格式，本工具负责转成标准 RVC pth |
| TTS | Text To Speech，文本转语音 |
| stems | 分离后的音轨（人声/鼓/贝斯/其他） |
| job | Gateway 任务；产物目录以 job id 命名 |
| profile | 内核的插件配置目录 |
| feed | 组件清单 JSON，带签名和 sequence |
| SSE | Server-Sent Events，任务进度推送 |
| data | 安装根下所有 AMS 数据的统一目录 |

---

## 17. 2026-10-24 维护修复

本版相对最初说明书有以下维护修复：

- 启动器状态轮询从 5 秒改为 10 秒，并移除了轮询中的自动音源扫描，解决点击/翻页卡顿。
- 内核/Gateway 停止消息不再拼接 Windows taskkill 的本地代码页输出，避免中文乱码。
- 组件下载增加字节级进度条（当前组件、百分比、已下载/总量）。
- 插件 Host 增加同源 loopback 认证兜底，解决某些 Chromium/WebView 环境丢失本地会话 Cookie 后 `/ams/api/summary`、`/ams/api/connection` 返回 401 的问题。
- 公开 feed 更新到 sequence 2038；启动器、Gateway、插件包、VC 服务同步更新。
- 新增“叠加系统声音”：麦克风与 WASAPI loopback 同时混音后再进入 RVC。
- 启动器新增内核 Web 端口黑名单：命中 6679 等 Node 禁用端口会自动杀掉重试。
- VC 停止后等待 250 ms 释放音频端点；快速重启遇到 -9999/-9997 会自动重初始化并重试。
- 旧版页面提交的 WDM-KS/MME 设备索引会自动映射到对应 WASAPI 端点，避免 WdmSyncIoctl 错误。
- Stage B MVP：独立 VC 服务、麦克风/立体声混音输入、RVC 实时推理、VB-Cable 输出、插件“实时变声”页。
- TTS 模型列表按名称去重：用户 data/tts_voices 中的同名模型优先，不再和 runtime 内置模型重复显示。
- 启动器增加“使用说明 / 关于 AMS”内部说明页。
- Stage A：data 布局自动补齐、TTS 用户音源目录 `data/tts_voices`、歌曲换源改为 RVC 纯人声、MSVC 独立为工具、打开目录按钮、RVC/pymss 必填与默认值修复。
- 交互热修：文件字段增加“选择文件”按钮并通过 Host 上传到 `data/projects/imports` 后回填路径；修复 Explorer 启动参数，目录按钮旁边显示固定 data 路径。

---

## 结束语

这份说明书按 2026-10-24 的公开版本编写：Launcher 0.2.0、feed 2042、插件 0.3.0、Gateway 0.1.0-stageG2。

## 19. 阶段 D 数据集与音源炼制

- 数据集工作室：dataset.prepare 导入 wav/目录，统一 48k 单声道，切成 3–10 秒片段，写 data/datasets/<名称>/dataset.json；
- train.rvc：数据集 → 预处理 → F0 → HuBERT 特征 → 训练 → FAISS index → 注册音色；
- 默认 v2/48k；40k 仅 v1（runtime 没有 v2 40k 配置）；
- 训练产物写入 data/voices/<名称>/，同名已存在会拒绝覆盖；
- 训练中的实验在 data/runtime/rvc/0.1.0/logs/ams_train_<job_id>；
- 失败/取消不注册半成品。

## 20. 阶段 E 歌曲换源完整化

- 分离任务行支持“用于换源”，自动填入 vocals；
- 换源任务行支持试听、A/B、导出纯人声；
- 换源页支持 batch_paths，每行一个纯人声文件批量提交；
- 导出目录：data/exports/songs；
- 超过 600 秒自动 300 秒分段，50ms 交叉淡化；12:09 实测输出 729.0 秒（长度锁定）；
- 纯人声链路不混伴奏。

## 18. 阶段 C 收尾

- RVC/pymss 全参数已开放；
- 参数模板保存/应用/导入/导出；删除移至 data/trash；
- 音色库支持标签、备注、收藏、index 绑定/解绑、试听、voicepack 导入/导出；
- TTS 支持 20000 字自动分段拼接、批量多模型；任务表可试听音频；
- MSVC 独立页签；
- 参数 schema 位于 data/config/schemas；
- 所有写入仍在 data 下。

如果你按第 4、5 章顺序安装，通常可以在 20–40 分钟内（取决于网速）得到一个可用的 TTS + RVC + pymss + MSVC 工作站。训练、项目管理、混音等页面如果当前不存在，不是安装失败，而是尚未实现；请以第 14 章为准。
