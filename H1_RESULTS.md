# H1 安装/生命周期阶段结果

日期：2026-10-25  
状态：**H1 未通过，不能进入 H2**。

## 已完成

1. setup 重建：2,438,343 字节，SHA256 D2DEFF5125CD857F41D9CE7A7C689E79A1DDA88AADA55F486DF4AE6060338B45。
2. 生成 CleanTest 安装器：productName AMS Studio CleanTest，identifier com.yanmc.ams.cleantest。
3. feed 2043：已签名，service.gateway version 修正为 0.1.0，plugin 0.3.0，已上传 ams-components.json / RELEASE_MANIFEST.json。
4. 版本清单：09_发布/_阶段G_20261025_135440/RELEASE_MANIFEST.json。
5. 源码 clean 检查通过。

## 未通过项

### CleanTest 安装路径错误

执行：AMS Studio CleanTest_0.2.0_x64-setup.exe /S /D=F:\AMS-Studio-CleanTest

结果：F:\AMS-Studio-CleanTest 为空；实际安装到 C:\Users\28988\AppData\Local\AMS Studio CleanTest，违反 AMS 数据应在安装目录/F 盘的原则。

按安全规约，未删除或移动 C 盘测试目录，留下待人工确认。

### 生产安装

- 生产 setup 此前是原地升级；
- 尚未做真正的干净目录安装；
- 未执行卸载；
- 未验证卸载保留 data。

### Kernel 生命周期

- 当前会话 watchdog 存在；
- 不是系统级常驻；
- 未做崩溃/重启专项验收。

## H1 结论

H1 未完成。已解决：setup 可重建；feed 版本元数据修正；发布清单建立。

阻断：/D 未生效，CleanTest 装到 C；干净安装未通过；卸载未实测；Kernel 常驻未落地。

## 下一步

不得进入 H2。先修正安装目录策略：给测试安装器显式设置 F 盘安装目录，或提供 portable/clean-root 安装模式；不再使用会落到 C 的默认 currentUser 安装；完成干净安装、覆盖升级、卸载保留 data 后，才可关闭 H0-D001/D002/D003。