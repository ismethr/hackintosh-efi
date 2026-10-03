# OpenCore EFI：Intel Core i7-12700 + Radeon RX 5700 XT

这是当前自用黑苹果配置的公开快照，适用于这台特定主机，不是通用 EFI。配置基于 OpenCore 1.0.8，并已移除 SMBIOS 身份信息、网卡 MAC 地址、磁盘布局、Windows BCD 和系统截图等不适合公开的内容。

> [!IMPORTANT]
> 仓库中的 `config.plist` **不能直接用于登录 Apple 服务**。使用前必须生成并填入你自己的 `SystemSerialNumber`、`MLB`、`SystemUUID` 和 `ROM`。不要把填好后的私人配置再次提交到公开仓库。

## 当前平台

| 项目 | 配置 |
| --- | --- |
| CPU | 12th Gen Intel Core i7-12700（20 线程） |
| 主板 | MAXSUN MS-Terminator B760M D4（铭瑄终结者 B760M D4） |
| 显卡 | AMD Radeon RX 5700 XT 8 GB |
| 内存 | 64 GB DDR4 |
| 有线网卡 | Realtek RTL8125B 2.5 GbE |
| 无线与蓝牙 | Intel AX210，macOS 下不使用，不加载驱动 |
| 音频 | 板载 Realtek 在 macOS 26 下不可用（系统已移除 AppleHDA），声音走显示器 DP 音频 |
| SMBIOS | MacPro7,1 |
| macOS | macOS 26.7.1（25G241） |
| OpenCore | 1.0.8 RELEASE |

主板型号取自 OpenCore 暴露的固件 OEM 信息。

## 目录说明

```text
EFI/
├── BOOT/
│   └── BOOTX64.efi            # OpenCore 1.0.8 官方回退启动入口
└── OC/
    ├── ACPI/                  # 当前机器使用的 SSDT
    ├── Drivers/               # OpenCore UEFI 驱动
    ├── Kexts/                 # macOS 驱动与补丁
    ├── Resources/             # OpenCanopy 主题资源
    ├── OpenCore.efi
    └── config.plist           # 已脱敏公开版
```

仓库不再包含 `EFI/Microsoft`。双系统用户应保留自己 EFI 分区里原有的 Microsoft 目录，不要用本仓库整目录覆盖现有 EFI 分区。

## ACPI 与驱动

启用的 SSDT：

- `SSDT-AWAC.aml`
- `SSDT-EC-USBX-DESKTOP.aml`
- `SSDT-GPRW.aml`
- `SSDT-PLUG-ALT.aml`

启用的 UEFI 驱动：

- `OpenCanopy.efi`
- `OpenHfsPlus.efi`
- `OpenRuntime.efi`
- `ResetNvramEntry.efi`
- `ToggleSipEntry.efi`

## Kext 版本

| Kext | 版本 | 用途 |
| --- | ---: | --- |
| Lilu | 1.7.2 | 补丁框架 |
| CpuTopologyRebuild | 2.0.2 | Alder Lake 大小核拓扑重建 |
| NVMeFix | 1.1.3 | 第三方 NVMe 电源管理 |
| VirtualSMC | 1.3.8 | SMC 模拟 |
| SMCProcessor | 1.3.8 | CPU 传感器 |
| SMCSuperIO | 1.3.8 | Super I/O 传感器（Nuvoton NCT6796D-E） |
| WhateverGreen | 1.7.1 | AMD 显卡相关补丁 |
| CPUFriend | 1.3.0 | CPU 电源管理 |
| CPUFriendDataProvider | 1.0.0 | 本机 CPUFriend 数据 |
| RestrictEvents | 1.1.6 | 机型与系统兼容补丁 |
| LucyRTL8125Ethernet | 1.2.3 | RTL8125B 2.5 GbE |
| RadeonSensor | 0.3.3 | AMD 显卡传感器 |
| SMCRadeonGPU | 0.3.3 | AMD 显卡 SMC 传感器桥接 |
| USBToolBox | 1.2.0 | USB 映射框架 |
| UTBMap | 1.1 | 当前主机 USB 端口数据 |

已移除 AppleALC、itlwm、IntelBluetoothFirmware、IntelBTPatcher 和 BlueToolFixup。macOS 26 已经没有 AppleHDA，AppleALC 无法工作；这台机器在 macOS 下也不使用 Wi-Fi 和蓝牙。

## 关键配置

- 启动参数：`agdpmod=pikera`
- `SecureBootModel = Disabled`，配合 RestrictEvents 的 `revpatch=sbvmm` 接收系统 OTA 更新
- `Vault = Optional`
- `ScanPolicy = 0`
- `UpdateSMBIOSMode = Custom`
- `CustomSMBIOSGuid = true`
- `LauncherOption = Full`，让 OpenCore 保持在固件启动项首位
- SIP：`csr-active-config = 40000000`，只关闭 NVRAM 保护，其余保护保持启用
- `EnableWriteUnprotector = false`：固件提供 Memory Attributes Table，由 `RebuildAppleMemoryMap` 和 `SyncRuntimePermissions` 接管
- 内核补丁 `Disable RTC wake scheduling` 与 `DisableRtcChecksum = true`：避免 BIOS 被重置或自动开机，代价是定时唤醒不可用
- `DeviceProperties` 为 RX 5700 XT 注入了自定义 `PP_PhmSoftPowerPlayTable`
- 图形启动器使用 OpenCanopy 自定义主题 `Blackosx/BsxM1`

`UpdateSMBIOSMode = Custom` 与 `CustomSMBIOSGuid = true` 是当前双系统策略的一部分，目的是减少 OpenCore SMBIOS 信息对 Windows 的影响。

## 使用前必须完成

1. 完整备份当前可启动的 EFI 分区，并准备可恢复的 U 盘 EFI。
2. 使用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 为 `MacPro7,1` 生成一套只属于你的参数。
3. 在 `EFI/OC/config.plist -> PlatformInfo -> Generic` 中替换以下占位值：
   - `SystemSerialNumber = REPLACE_ME`
   - `MLB = REPLACE_ME`
   - `SystemUUID = 00000000-0000-0000-0000-000000000000`
   - `ROM = 00:00:00:00:00:00`
4. 双系统机器只合并 OpenCore 文件，不要删除或覆盖原有 `EFI/Microsoft`。
5. 首次测试从 U 盘或备用启动项进入；确认启动、网络、USB、睡眠和显卡加速后，再替换主 EFI。
6. 更改 SMBIOS 或重要 NVRAM 配置后，在 OpenCore 启动菜单执行一次 Reset NVRAM。

关于四项 SMBIOS 字段的对应关系，可参考 [Dortania PlatformInfo 指南](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html#platforminfo)。

## 已知维护项

- USB 已用 USBToolBox 完成映射：`UTBMap` 保留 15 个端口（HS01–HS09、SS01–SS06），`XhciPortLimit = false`。
- 如需板载音频，要另找方案（例如 VoodooHDA），本仓库不包含。
- `CPUFriendDataProvider`、`UTBMap`、SSDT、显卡 PowerPlay 表和设备路径都针对当前硬件，换主板、CPU、显卡或 USB 布局后必须重新制作。
- OpenCore、kext 或 macOS 大版本升级前，应先在备用 EFI 上验证，避免直接覆盖当前可启动配置。

## 隐私与发布版说明

本次公开版明确排除了：

- SMBIOS 序列号、MLB、SystemUUID、ROM
- 网卡 MAC 地址、磁盘分区名称与布局
- Windows BCD、恢复数据和其他 Microsoft 启动文件
- 系统截图、日志、NVRAM 导出
- `.DS_Store`、AppleDouble `._*` 等 macOS 元数据

公开仓库只应保存脱敏模板。私人可启动版请单独离线备份，不要提交。

## 上游项目

- [OpenCorePkg](https://github.com/acidanthera/OpenCorePkg)
- [OcBinaryData](https://github.com/acidanthera/OcBinaryData)
- [Lilu](https://github.com/acidanthera/Lilu)
- [WhateverGreen](https://github.com/acidanthera/WhateverGreen)
- [VirtualSMC](https://github.com/acidanthera/VirtualSMC)
- [NVMeFix](https://github.com/acidanthera/NVMeFix)
- [CpuTopologyRebuild](https://github.com/b00t0x/CpuTopologyRebuild)
- [LucyRTL8125Ethernet](https://github.com/Mieze/LucyRTL8125Ethernet)
- [USBToolBox](https://github.com/USBToolBox/kext)

## 免责声明

Hackintosh 配置与主板 BIOS、固件版本和具体硬件批次强相关。使用前请自行核对并保留恢复手段；本仓库仅作为个人配置记录与参考。
