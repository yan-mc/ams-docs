# AMS-Test VM 搭建与部署测试记录

> 时间：2026-10-20 23:05｜所有 VM 数据在 F 盘；宿主机只启用 Hyper-V 系统功能（系统组件在 C 盘不可避免）。

## 1. 结果

- Hyper-V 已启用并重启生效（vmms/HvHost Running，HypervisorPresent=True）；
- VM `AMS-Test`：8 vCPU / 16GB / 120GB 动态盘 / 第 2 代 / Secure Boot 关闭（安装镜像兼容）/ vTPM 已配置；
- Guest：Windows 11 专业版 Build 26300，已安装成功并自动创建管理员 `amsdev`；
- Guest 盘：C: 119.7GB（剩余 93.9GB）；D: 为 Windows 安装 ISO；
- AMS Studio Launcher **0.2.0 已在 Guest 内静默安装成功**：
  - `C:\Users\amsdev\AppData\Local\AMS Studio\ams-launcher.exe`（6,714,368 B）；
  - 卸载项 `AMS Studio 0.2.0`；
  - `C:\Users\amsdev\.dsh` 不存在，未触碰任何 DSH 数据。
- 交互式启动测试：通过计划任务以 `amsdev` Interactive 身份启动，进程 `ams-launcher` 存活；非交互 PowerShell Direct 直接启动 GUI 会崩溃（0xc0000409），这是 Session 0 限制，不是产品缺陷。
- 已创建检查点 `baseline-m2-clean`（干净 Windows + launcher 0.2.0）。

## 2. 关键路径

| 项 | 路径 |
|---|---|
| VM 配置 | `F:\AMS-VM\Hyper-V` |
| 虚拟磁盘 | `F:\AMS-VM\VHDX\AMS-Test.vhdx`（约 16.7GB） |
| 检查点 | `F:\AMS-VM\Snapshots` |
| ISO | `F:\AMS-VM\ISO` |
| 脚本 | `F:\AMS-VM\Scripts` |
| 日志/截图 | `F:\AMS-VM\Logs` |
| Guest 账号 | `amsdev` / `AmsVM!2026`（仅本机测试） |

## 3. 安装过程教训

- Windows ISO 的 UEFI 启动需要按键；UEFI 启动顺序把网络放在前面时会先 PXE。最终采用 **宿主机 DISM 直接 apply install.wim index 4 + bcdboot**，绕开交互式 Setup；
- 离线应用 unattend 成功（`dism /Apply-Unattend`），Guest 自动创建 amsdev；
- 不能用 PowerShell Direct 的 Session 0 直接判断 GUI 应用；GUI 验证必须用 Interactive 计划任务或 vmconnect；
- 中文路径的 .ps1 必须带 UTF-8 BOM，否则 Windows PowerShell 5.1 会乱码。

## 4. 下一步（在 VM 内，禁止宿主机安装 AMS）

1. 打包 **独立 AMS kernel**（fork DSH + 自带 Node/Electron 运行时 + 私有 profile）；
2. 启动器 `start_kernel` 清空继承的 `DSH_*`/`NODE_*`/`ELECTRON_*`，使用安装目录内绝对路径，设置独立 `AMS_HOME`/`DSH_HOME`/`profile=music`；
3. 在 VM 内同时运行现有 DSH 与 AMS，验证：不共用 `.dsh`、不共用端口、不互相单实例、不互相杀进程；
4. 验证通过前不发布；GPU 功能（RVC/pymss/实时变声）留宿主机测试。
