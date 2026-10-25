# H1 安装系统重写与验收（2026-10-25）

状态：**H1 通过**。

## 1. 新安装系统

旧 Tauri NSIS 模板已废弃，改为独立 NSIS 安装器：

- installer/ams-setup.nsi
- installer/build-installer.ps1
- 安装器产物：AMS-Studio-Setup-0.2.1.exe
- 字节：3,125,583
- SHA256：F6924345F5064529ADD5678DD72CC9ADF2A3757E921FA1B65BEAE42E486E6C05

设计：

- 默认安装到 F:\AMS-Studio；
- 支持 /D 指定目录；
- 支持 /S 静默安装；
- 只安装 launcher 和 data 目录骨架；
- 覆盖升级只替换程序文件；
- 卸载只删除 launcher 和 uninstaller；
- data 永不递归删除；
- 不使用 Tauri NSIS 模板、不使用 currentUser AppData 路径。

## 2. 干净安装

目标：F:\AMS-Studio-H1Final

命令：

    AMS-Studio-Setup-0.2.1.exe /S /D=F:\AMS-Studio-H1Final

结果：

- install_exit=0
- launcher 存在；
- data 存在；
- 未写入 C 盘 AMS 数据。

## 3. 卸载保留 data

写入 data\H1_FINAL_KEEP.txt 后执行：

    AMS-Studio-Uninstall.exe /S

结果：

- uninstall_exit=0
- launcher 被删除；
- data 标记保留；
- data 目录保留。

## 4. 生产覆盖升级

目标：F:\AMS-Studio

执行：

    AMS-Studio-Setup-0.2.1.exe /S

结果：

- launcher 更新到 0.2.1；
- data\H1_UPGRADE_KEEP.txt 保留；
- data\voices\曼波\曼波.pth 的 SHA256 安装前后一致；
- launcher 可启动；
- Kernel 进程启动。

## 5. Kernel 恢复

停止全部 AMS Kernel 后：

- watchdog 约 25 秒内重启 Kernel；
- 新进程组出现。

当前策略：launcher 启动 Kernel + 会话 watchdog 自动恢复；系统级服务仍留到发布阶段。

## 6. C 盘清理

逐文件删除 CleanTest 的两个程序文件，删除空目录和仅属于 CleanTest 的 HKCU 注册项。未触碰生产数据。

## 7. feed/setup 统一

feed：2044

setup：

    AMS-Studio-Setup-0.2.1.exe
    3,125,583 bytes
    F6924345F5064529ADD5678DD72CC9ADF2A3757E921FA1B65BEAE42E486E6C05

feed 中的 setup 字段与新安装器一致，setup 资产已上传 GitHub v0.2.1。

## 8. 已知边界

- 本次干净安装是 F 盘干净目录，不是独立 Windows VM；环境没有 Windows Sandbox。
- 系统级 Kernel 服务仍为后续发布阶段工作。
- 测试 data 和测试工程保留在 F 盘，尚未做逐文件清理。

## 9. 结论

H1 通过。

安装、升级、卸载保留 data、Kernel 恢复、版本/setup/feed 一致化均完成。

下一阶段：H2 功能回归。
