# 音源炼制问题修复记录（2026-10-25）

## 原问题

- 50 epoch 训练在 UI 显示 69.6% 停留约 1 小时；
- 最终任务状态 failed，错误为 Gateway BACKEND_TIMEOUT 7200s；
- 但训练实际完成，e10/e20/e30/e40/e50、D/G、最终 pth/index/voice 均已生成；
- GPU 占用忽上忽下：训练/保存 checkpoint/FAISS index 阶段正常切换。

## 根因

1. RVC 子进程 stdout 缓冲，Epoch 进度不能实时到达；
2. 训练结束后 index/注册阶段没有独立进度；
3. Gateway 训练超时 7200s 小于 538 clips/50 epoch 的实际耗时；
4. 超时在最后注册阶段触发，导致产物已生成但任务 failed。

## 修复

源码：

- app/runners/train.py：timeout 7200 -> 21600 秒；
- app/runners/scripts/rvc_train.py：
  - RVC 子进程加 -u，实时 stdout；
  - train 结束显式 85%；
  - index 写入后 92%；
  - voice 注册后 98%；
  - done 100%。

部署：

- 重建 service.gateway；
- 当前 Gateway 5625 已运行修复版；
- 旧组件备份：F:/AMS-Studio/data/backups/service.gateway-trainfix-old。

## 验证

小数据集 20 clips，1 epoch：

    job_2d52aadff0fc4a22
    status: succeeded
    progress: 50 -> 84 -> 88 -> 100
    pth: fix_smoke.pth
    index: fix_smoke.index

旧的闲云完整产物仍可用：

    F:/AMS-Studio/data/voices/闲云/闲云.pth
    F:/AMS-Studio/data/voices/闲云/闲云.index
    RVC job_8b6b91e3d670423f succeeded，9.3973s/48k

## 结论

- 训练不再在收尾时被 2 小时硬超时误杀；
- 进度能进入 index/register 阶段；
- 旧闲云最终音源不重训也可用；
- 后续长训练建议仍按数据量评估超时（当前 6 小时）。
