# H3 RC 构建与干净目录验收记录（2026-10-25）

状态：**H3 非 H2 依赖工作已完成；RC 已构建并草稿上传，尚未发布。**

## 1. RC 版本

1.0.0-rc.1

| 组件 | 版本 |
|---|---|
| Launcher | 1.0.0-rc.1 |
| Plugin | 1.0.0-rc.1 |
| Gateway | 1.0.0-rc.1 |
| Installer | 1.0.0-rc.1 |
| Kernel | 0.2.1-rc.2（上游映射） |
| feed | 2045 |

## 2. RC 发布物

目录：

F:/Deepseek_harness/AI音乐处理/09_发布/_H3_1.0.0-rc.1

文件：

| 文件 | 字节 |
|---|---:|
| AMS-Studio-Setup-1.0.0-rc.1.exe | 3,089,586 |
| service-gateway-1.0.0-rc.1.zip | 16,809,831 |
| plugin-ams-music-bundle-1.0.0-rc.1.zip | 45,852 |
| ams-components.json（feed 2045，已签名） | 8,085 |
| RELEASE_MANIFEST.json | 906 |
| SHA256SUMS.txt | 390 |

setup SHA256：

887C04A06F749C0DEBFB6EC16DC3FE4F1A1FFF56CFBFA700A34BAFABB93370F4

feed SHA256：

863ebd9facb0c640e417b8ffad9a891dae53197d35f7495f552cc841309fb11b

## 3. 发布状态

已创建 GitHub draft prerelease：

tag: v1.0.0-rc.1
draft: true
prerelease: true
release id: 423437440

已上传：

- ams-components.json
- AMS-Studio-Setup-1.0.0-rc.1.exe
- service-gateway-1.0.0-rc.1.zip
- plugin-ams-music-bundle-1.0.0-rc.1.zip
- RELEASE_MANIFEST.json
- SHA256SUMS.txt

未发布，因此不影响当前 H2 latest（feed 2044/0.2.1）。

## 4. QA 结果

QA 入口：

F:/Deepseek_harness/AI音乐处理/08_项目代码/ams-release/qa/run-h3.ps1

最近结果目录：

F:/AMS-Studio/data/qa/H3/20261025-215933

通过：

- 版本统一 1.0.0-rc.1
- feed 2044 与当前 setup 一致
- release manifest 2044 一致
- Gateway pytest
- release manifest pytest
- v44 feed 签名
- RC feed 2045 与 RC manifest 一致
- RC 签名
- RC setup hash

未自动化：

- UI CDP smoke
- installer clean-directory
- clean-dir RC

安装器/干净目录已手工验收。

## 5. 干净目录 RC 验收

安装：

F:/AMS-Studio-RC-Clean
setup 1.0.0-rc.1
/S /D=F:/AMS-Studio-RC-Clean

结果：

- 安装成功
- launcher FileVersion 1.0.0-rc.1
- data 目录生成
- 无 C 盘 AMS data

卸载保留 data：

- launcher 删除
- data 保留
- H3_RC_KEEP.txt 保留

升级：

F:/AMS-Studio-H3Upgrade
0.2.1 -> 1.0.0-rc.1

- 旧 launcher 0.2.1
- RC 覆盖后 1.0.0-rc.1
- data marker 保留

回滚：

RC 1.0.0-rc.1 -> 0.2.1

- launcher 回到 0.2.1
- data marker 保留
- 卸载后 marker 保留

## 6. H3 剩余（需 H2 完成后）

- H2 闲云 50 epoch 训练完成
- H2 最终中文 RVC/评测
- H3 RC 的 UI CDP 在 RC 构建上验收
- RC draft -> published
- 旧 GitHub 资产保留/归档
- INSTALLATION/RC 发布说明最终更新
- P0/P1 缺陷清零
- H3 签收

## 7. 说明

- 虚拟机已彻底拆除，VM 不再作为干净环境；H3 使用干净目录/便携 RC，必要时用独立物理机。
- H2 当前训练使用已安装的 0.2.1 Gateway；本次 RC 构建未替换、未重启 H2 Gateway。