# AMS Studio H3 RC 安装/升级/回滚说明

适用版本：AMS Studio 1.0.0-rc.1  
feed：2045  
setup：AMS-Studio-Setup-1.0.0-rc.1.exe

## 安装

1. 关闭旧版 Launcher/Kernel。
2. 运行 setup：
   - 默认安装目录 F:/AMS-Studio；
   - 可用 /D 指定目录。
3. 安装器只写程序文件和 data 目录骨架，不写 C 盘 AMS data。
4. 启动 Launcher，安装/校验组件，启动 Kernel 和 Gateway。
5. 浏览器打开 AMS Kernel，点击左侧“AMS 音乐”。

## 升级

1. 备份 data/voices、data/projects、data/config。
2. 直接运行 RC setup 覆盖安装。
3. 安装器保留 data；只替换 launcher/uninstaller。
4. 启动后检查 voices/projects/config。

## 回滚

1. 关闭 Launcher/Kernel。
2. 运行上一版 0.2.1 setup 覆盖安装。
3. data 不动。
4. 启动旧版并检查音色/工程。

## 卸载

1. 运行 AMS-Studio-Uninstall.exe。
2. 只删除程序文件。
3. data 保留，不递归删除。

## 已验证

- 干净目录安装：通过；
- 0.2.1 -> RC 升级：通过；
- RC -> 0.2.1 回滚：通过；
- 卸载保留 data：通过；
- 无 C 盘 AMS data。

## 已知边界

- 虚拟机无法调用 RTX 4080，H3 不使用 VM；干净环境使用干净目录/便携 RC 或独立物理机。
- 中文闲云 50 epoch 训练完成前，RC 不作为最终发布。
- GitHub 旧资产尚未归档。
- 旧 INSTALLATION.md 仍含 0.2.0 内容，H3 发布前更新。

## 校验

setup SHA256：

21D0C12F3AB00EBAAB1DDDA22AB88E342CA7508E6FF113866A39C81AF9922BE8

feed 2045 签名已验证。