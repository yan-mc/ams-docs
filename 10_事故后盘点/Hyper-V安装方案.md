# Hyper-V 安装与隔离开发方案（AMS Studio）

> 适用机器：MECHREVO 旷世16 Super（i9-13900HX / 32GB / RTX 4080 Laptop 12GB）。
> 系统：注册表显示 Windows 10 Pro / Build 26100（24H2 内核）；虚拟化固件已开启、SLAT 支持；**Hyper-V 当前未启用**。
> 本文只是方案与步骤，**没有启用 Hyper-V、没有下载 ISO、没有创建 VM**。

## 0. 先明确边界

- **VM 适合**：第二个 DSH / fork 内核 / 插件 / 启动器 / 安装器 / 危险清理脚本 / 文件删除类实验；快照回滚。
- **VM 不适合**：RVC、pymss、任何需要 CUDA 的推理/训练。消费级 RTX 4080 Laptop 不能直通给 Hyper-V/VMware/VirtualBox；Hyper-V GPU-P 只支持特定企业 GPU/Windows Server，不适用于这台机器。
- 推荐“**宿主做 GPU，VM 做 DSH/构建/危险测试**”；音源和模型用只读共享或复制测试夹具。

## 1. 硬件与资源建议

| 项 | 建议 | 说明 |
|---|---|---|
| vCPU | 8 | 宿主 24C/32T，留足够给宿主 |
| 内存 | 16GB（固定，不启用动态内存） | 宿主 32GB；Tauri/Electron 构建足够 |
| 磁盘 | 120GB 动态 VHDX，放 D: 或 E: | D 剩 398GB / E 剩 559GB；不要放 C: |
| 网络 | Default Switch（NAT）或 Internal + NAT | 不桥接宿主内网 |
| GPU | 无（WDDM 基本显示） | GPU 测试留在宿主 |
| 快照 | 基线快照 + 危险操作前快照 | 标准检查点即可 |
| 系统 | Windows 11/10 Pro x64 官方 ISO | 需要单独授权 |

## 2. 启用 Hyper-V（需管理员 + 重启）

```powershell
Get-ComputerInfo | Select-Object WindowsProductName,WindowsVersion,OsBuildNumber
(Get-CimInstance Win32_ComputerSystem).HypervisorPresent
(Get-CimInstance Win32_Processor | Select-Object -First 1).VirtualizationFirmwareEnabled
```

启用（PowerShell 管理员）：

```powershell
# 图形：控制面板 → 程序 → 启用或关闭 Windows 功能 → 勾选 Hyper-V → 重启
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All -All -NoRestart
# 重启后验证
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
Get-Service vmms
(Get-CimInstance Win32_ComputerSystem).HypervisorPresent
```

如果提示功能未知/需要专业版：Home 版没有官方 Hyper-V，改用 VMware Workstation Pro 或 VirtualBox。

> 启用 Hyper-V 会与部分旧版安卓模拟器/VMware/VirtualBox 冲突；本机目前没有这些软件。

## 3. 准备 Windows ISO

- 微软官网下载 Windows 11 x64 ISO（约 6–7GB）；
- 校验 SHA256（官网提供）；保存到 `D:/ISO/` 或 `E:/ISO/`；
- 不使用来源不明的精简版/修改版。

## 4. 创建虚拟机

### 4.1 Hyper-V 管理器（图形）

1. 打开 Hyper-V 管理器；
2. 新建 → 虚拟机；名称 `AMS-Test`；代数 **第 2 代**；
3. 启动内存 **16384MB**，取消“使用动态内存”；
4. 网络选 `Default Switch`；
5. 创建虚拟硬盘 **120GB**（动态扩展），位置 `D:/Hyper-V/AMS-Test/`；
6. 安装选项：从可启动 CD/DVD 安装，选择 ISO；
7. 完成 → 连接 → 启动 → 安装 Windows。

### 4.2 PowerShell（管理员）

```powershell
$vm = "AMS-Test"
$dir = "D:/Hyper-V/AMS-Test"
New-Item -ItemType Directory -Force -Path $dir | Out-Null
New-VM -Name $vm -MemoryStartupBytes 16GB -Generation 2 -NewVHDPath "$dir/$vm.vhdx" -NewVHDSizeBytes 120GB -SwitchName "Default Switch"
Set-VMProcessor -VMName $vm -Count 8
Set-VMMemory -VMName $vm -DynamicMemoryEnabled $false
Add-VMDvdDrive -VMName $vm -Path "D:/ISO/Win11.iso"
$dvd = Get-VMDvdDrive -VMName $vm
Set-VMFirmware -VMName $vm -FirstBootDevice $dvd
Start-VM -Name $vm
vmconnect.exe localhost $vm
```

> 如果 `Default Switch` 名称不同，先 `Get-VMSwitch` 查看。

## 5. 安装 Guest 系统

- 语言/键盘按需；安装类型“自定义”；选 120GB 虚拟盘；
- 安装完成后跑 Windows 更新；Hyper-V 集成服务对 Windows Guest 内置；
- 设置本地管理员密码；创建专用本地账号 `amsdev`（不要用宿主微软账号登录）；
- 可选省空间：`powercfg /h off`；
- 确认网络：`ipconfig`、浏览器能打开 GitHub/微软官网；
- 安装 7-Zip/常用工具（可选）。

## 6. 网络：Default Switch 或 Internal + NAT

**Default Switch**：最简单，VM 自动 NAT 上网；宿主重启后 IP 可能变化。

**Internal + NAT（更隔离，管理员）**：

```powershell
New-VMSwitch -Name "AMS-Internal" -SwitchType Internal
$if = Get-NetAdapter | Where-Object Name -like "*AMS-Internal*"
New-NetIPAddress -InterfaceIndex $if.ifIndex -IPAddress 192.168.100.1 -PrefixLength 24
New-NetNat -Name "AMS-NAT" -InternalIPInterfaceAddressPrefix "192.168.100.0/24"
```

Guest 内设置：IP `192.168.100.10`，网关 `192.168.100.1`，DNS `8.8.8.8`/`1.1.1.1`。

> 不要桥接宿主 WLAN；不要给 VM 访问宿主 C: 的共享。传文件用 ISO/第二块 VHDX/RDP 驱动器重定向。

## 7. Guest 内开发环境

1. Git for Windows；
2. Node.js LTS 24；
3. `corepack enable` + `corepack prepare pnpm@11.28.5 --activate`；
4. Rustup + stable MSVC 工具链；
5. Visual Studio Build Tools 2022，勾选“使用 C++ 的桌面开发”（MSVC + Windows SDK）；
6. WebView2 Runtime（Win11 通常自带；Tauri 需要）；
7. Python 3.12（Gateway 开发需要）；
8. 7-Zip（可选）。

VS Build Tools 静默示例（Guest 管理员）：

```powershell
.\vs_BuildTools.exe --quiet --wait --norestart --add Microsoft.VisualStudio.Workload.VCTools --includeRecommended
```

验证：`node -v; pnpm -v; rustc --version; cargo --version`。

## 8. 把项目放进 Guest

1. **Guest 内 git clone 私有仓库**（推荐）：Guest 生成 SSH key 或 fine-grained PAT，只授权 7 个 AMS 仓库；宿主 token 不进 Guest；
2. **git bundle 离线复制**：宿主 `ams-release/tools/backup.ps1` 生成 bundle → 通过 ISO/第二 VHDX 复制到 Guest → clone。

不要把宿主 `C:/Users/28988/.dsh` 直接挂进 Guest；Guest 使用独立 `DSH_HOME`（如 `C:/Users/amsdev/.dsh`）。

## 9. Guest 内验证

```powershell
git clone git@github.com:yan-mc/ams-harness.git
cd ams-harness; pnpm install --frozen-lockfile; pnpm run build
cd ../ams-launcher; pnpm install --frozen-lockfile; pnpm tauri build
cd ../ams-plugins; pnpm install --frozen-lockfile; pnpm run build
```

注意：Guest 没有 CUDA，不要运行 RVC/pymss GPU；GPU 测试在宿主。

## 10. 快照与回滚

```powershell
Checkpoint-VM -Name "AMS-Test" -SnapshotName "baseline-dev"
Get-VMSnapshot -VMName "AMS-Test"
Restore-VMSnapshot -VMName "AMS-Test" -Name "baseline-dev" -Confirm:$false
```

建议：`baseline-os` → `baseline-dev` → 每次危险实验前 `pre-<任务>`。

## 11. 安全设置（重要）

- Guest 独立本地账号 / 独立 GitHub token / 独立 DeepSeek Key；
- 关闭增强会话剪贴板/驱动器重定向；
- 不共享宿主 C:；不把宿主 `.dsh`、`.credentials.yaml` 复制进 Guest；
- Guest 内遵守“只写工作区”；清理脚本先在快照里跑；
- 定期 push 到私有 GitHub；宿主保留 bundle 备份。

## 12. 常见问题

| 问题 | 处理 |
|---|---|
| `Enable-WindowsOptionalFeature` 找不到 Hyper-V | 确认 Pro/Enterprise；Home 用 VMware/VirtualBox |
| `vmms` 不存在 | Hyper-V 未启用成功，重启后复查 |
| Guest 不能上网 | 检查 Default Switch；Internal 需 NAT/DNS |
| 宿主与 Guest 抢资源 | VM 固定 16GB/8 vCPU；宿主 GPU 任务优先 |
| GPU 不能用 | 预期行为；不要直通消费级笔记本 GPU |
| 安装器冒烟测试 | 可先用 Windows Sandbox；完整持久开发用 Hyper-V |

## 13. 需要下载/授权的清单

| 项 | 大小/方式 | 是否必须 |
|---|---|---|
| Windows 11 x64 ISO（官方） | 约 6–7GB | 必须 |
| Guest 开发工具（Node/pnpm/Rust/VS BuildTools/WebView2） | 约 5–10GB | 必须 |
| Windows Guest 授权 | 按现有/单独授权 | 必须 |
| VMware Workstation Pro（备选） | 免费个人版，Broadcom 下载 | 可选 |
| VirtualBox（备选） | 约 100MB | 可选 |

**执行前请确认：允许下载 Windows ISO、启用 Hyper-V（会重启）、VM 放 D 还是 E。** 未确认前不要动系统功能。
## 14. F 盘落位（2026-10-20 实测）

- F: 是 **NTFS / 固定盘 / NVMe / 1TB**，剩余 **363.8GB**，适合放 Hyper-V 的 VM 配置、VHDX、快照、ISO；
- 已创建 `F:/AMS-VM/`：`Hyper-V/`（配置）、`VHDX/`（虚拟盘）、`Snapshots/`（检查点）、`Exports/`（导出）、`ISO/`、`Scripts/`；
- `Scripts/01-enable-hyperv.ps1`：启用 Hyper-V（需管理员 + 重启）；
- `Scripts/02-create-ams-vm.ps1`：自动在 F 盘创建 8 vCPU / 16GB / 120GB 动态盘的 `AMS-Test`，ISO 自动从 `F:/AMS-VM/ISO/` 查找；
- 说明：Hyper-V 的 **Windows 系统功能本体**仍在 C 盘（系统组件无法搬到 F），但所有 VM 数据/磁盘/快照/ISO 都在 F。

## 15. 开发工具 vs 最终软件依赖

| 工具 | 最终用户是否需要 | 说明 |
|---|---|---|
| Node.js / pnpm | **不需要**（用户侧） | 开发期构建前端/插件；最终内核是预编译 Electron/DSH 包，内部自带 Node 运行时；用户不装 Node |
| Rust / rustc / cargo | **不需要** | 只用于编译 Tauri 启动器；最终发 exe/msi，用户不装 Rust |
| VS BuildTools / MSVC | **不需要** | 只用于编译 Rust/C++；最终用户不装 VS |
| Python / torch | **运行依赖，随组件提供** | Gateway/RVC/pymss 属于后端；由 portable runtime/组件安装器提供，不需要用户系统 Python |
| WebView2 | 需要（Win11 通常内置） | Tauri 启动器渲染依赖；缺失时安装器提示/安装 |
| NVIDIA 驱动 | 需要（GPU 功能） | RVC/pymss GPU 才有；CPU 路径不需要 |

结论：**Node/Rust/VS BuildTools 都是开发机工具**。宿主机正常装 D 盘/C 盘都可以，不必搬进 F；当前已有的 VS BuildTools/Rust 留在 C 不影响最终软件。若在 VM Guest 内构建，Guest 的工具装在 Guest 的 VHDX 里（物理上也在 F）。
## 16. 执行状态（2026-10-20 22:15）

- Hyper-V 功能已通过管理员脚本启用，状态 **pending reboot**；尚未重启；
- Windows 11 ISO 已放到 `F:/AMS-VM/ISO/Windows11_Client_x64_zh-cn_26300_9457.iso`（8.49GB）；
- `F:/AMS-VM` 目录与两个脚本已就绪；尚未创建/启动 VM；
- 下一步：重启 Windows → 运行 `02-create-ams-vm.ps1`。
