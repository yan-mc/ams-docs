# H1 安装/生命周期阶段结果（2026-10-25）

状态：**H1 未完成，不能进入 H2。**

## 已完成

### 方案 A 自定义 NSIS 模板

- 自定义模板：src-tauri/nsis/ams-installer.nsi
- currentUser 分支不再写 LOCALAPPDATA；
- 默认安装到 F:\AMS-Studio；
- 支持 AMS_INSTALL_DIR 测试覆盖；
- CleanTest 安装器已构建。

### CleanTest 干净安装

执行 CleanTest 安装器，设置：

    AMS_INSTALL_DIR=F:\AMS-Studio-CleanTest

结果：

- 程序装到 F:\AMS-Studio-CleanTest；
- F 目标非空；
- 启动后生成 data；
- data 下建立 assets/cache/components/config/datasets/kernel/logs/voices 等目录；
- 没有写到 C 盘 AMS data。

### 卸载保留 data

在 CleanTest data 写入标记文件，运行 uninstall.exe /S：

- 程序文件被卸载；
- data/H1_KEEP.txt 仍存在；
- 说明 uninstaller 保留 data 的规则成立。

### C 盘遗留物清理

已逐文件删除：

    C:\Users\28988\AppData\Local\AMS Studio CleanTest\ams-launcher.exe
    C:\Users\28988\AppData\Local\AMS Studio CleanTest\uninstall.exe

删除空目录和两个仅属于 CleanTest 的 HKCU 注册项。未触碰生产 AMS Studio。

### Kernel 崩溃恢复

停止全部 AMS Kernel 后，watchdog 在约 25 秒内重启 Kernel；新进程组出现。当前会话崩溃恢复通过。

### 版本/feed

- feed 2043 已生成并签名；
- service.gateway version 修正为 0.1.0；
- plugin 0.3.0；
- 版本清单 RELEASE_MANIFEST.json 已生成。

## 未通过项

### 生产 0.2.1 覆盖安装没有更新 launcher

生产 setup：

    AMS Studio_0.2.1_x64-setup.exe

执行 /S 后 exit=0，但：

- F:\AMS-Studio\ams-launcher.exe 时间仍是 13:38；
- data/H1_UPGRADE_KEEP.txt 保留；
- 升级没有真正替换程序文件。

CleanTest 通过，但生产升级路径不通过。需要在 H1 内解决，不能进入 H2。

### 其他

- setup/feed 资产需要以 0.2.1 新 setup 统一更新；
- 干净 VM 仍未做（当前只做同机 F 盘隔离目录）；
- Kernel watchdog 仍为当前会话，不是系统服务。

## 结论

H1 关键突破：

- 自定义 NSIS 模板解决了默认 C 盘安装；
- CleanTest F 盘安装/启动/卸载保留 data 通过；
- C 遗留物已按授权逐文件清理；
- Kernel 崩溃恢复通过。

H1 阻断：

- 生产 0.2.1 覆盖安装 no-op；
- 生产升级未验证；
- setup/feed 0.2.1 资产未完全统一。

不得进入 H2。下一步集中排查生产安装器 no-op（运行中进程、旧 uninstaller、安装目录注册、升级模式），再用 0.2.1 完成生产覆盖升级。
