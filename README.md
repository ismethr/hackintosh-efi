# OpenCore EFI：Intel Core i7-12700 + Radeon RX 5700 XT

这是当前自用黑苹果配置的公开快照，适用于这台特定主机，不是通用 EFI。配置基于 OpenCore 1.0.7，并已移除 SMBIOS 身份信息、网卡 MAC 地址、磁盘布局、Windows BCD 和系统截图等不适合公开的内容。

> [!IMPORTANT]
> 仓库中的 `config.plist` **不能直接用于登录 Apple 服务**。使用前必须生成并填入你自己的 `SystemSerialNumber`、`MLB`、`SystemUUID` 和 `ROM`。不要把填好后的私人配置再次提交到公开仓库。

## 当前平台

| 项目 | 配置 |
| --- | --- |
| CPU | 12th Gen Intel Core i7-12700（20 线程） |
| 主板 | MAXSUN 桌面平台（具体型号未从当前系统可靠识别） |
| 显卡 | AMD Radeon RX 5700 XT 8 GB |
| 内存 | 64 GB DDR4 |
| 有线网卡 | Realtek RTL8125B 2.5 GbE |
| 无线与蓝牙 | Intel 方案，使用 `itlwm`、`IntelBluetoothFirmware` 等驱动 |
| SMBIOS | MacPro7,1 |
| macOS | macOS 26.5.2（25F84） |
| OpenCore | 1.0.7 RELEASE |

主板具体型号和 Intel 无线网卡型号暂未写入，避免根据不完整信息猜测。确认后可再补充。

## 目录说明

```text
EFI/
├── BOOT/
│   └── BOOTX64.efi            # OpenCore 1.0.7 官方回退启动入口
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
| WhateverGreen | 1.7.0 | AMD 显卡相关补丁 |
| AppleALC | 1.9.7 | 板载音频，当前 `alcid=12` |
| VirtualSMC | 1.3.7 | SMC 模拟 |
| SMCProcessor | 1.3.7 | CPU 传感器 |
| SMCSuperIO | 1.3.7 | Super I/O 传感器 |
| RadeonSensor | 0.3.3 | AMD 显卡传感器 |
| SMCRadeonGPU | 0.3.3 | AMD 显卡 SMC 传感器桥接 |
| CPUFriend | 1.3.0 | CPU 电源管理 |
| CPUFriendDataProvider | 1.0.0 | 本机 CPUFriend 数据 |
| LucyRTL8125Ethernet | 1.2.3 | RTL8125B 2.5 GbE |
| itlwm | 2.3.0 | Intel Wi-Fi；通常配合 HeliPort 使用 |
| IntelBluetoothFirmware | 2.5.0 | Intel 蓝牙固件 |
| IntelBTPatcher | 2.5.0 | Intel 蓝牙补丁 |
| BlueToolFixup | 2.6.9 | macOS 蓝牙框架兼容 |
| USBToolBox | 1.2.0 | USB 映射框架 |
| UTBMap | 1.1 | 当前主机 USB 端口数据 |
| RestrictEvents | 1.1.6 | 机型与系统兼容补丁 |

## 关键配置

- 启动参数：`agdpmod=pikera alcid=12`
- `SecureBootModel = Disabled`
- `Vault = Optional`
- `ScanPolicy = 0`
- `UpdateSMBIOSMode = Custom`
- `CustomSMBIOSGuid = true`
- SIP 保持启用
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
5. 首次测试从 U 盘或备用启动项进入；确认启动、网络、音频、USB、睡眠和显卡加速后，再替换主 EFI。
6. 更改 SMBIOS 或重要 NVRAM 配置后，在 OpenCore 启动菜单执行一次 Reset NVRAM。

关于四项 SMBIOS 字段的对应关系，可参考 [Dortania PlatformInfo 指南](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html#platforminfo)。

## 已知维护项

- 当前 USB 配置保留了 `XhciPortLimit = true`，`UTBMap` 中列出了 25 个端口。这是现状快照，不是理想的长期配置。后续建议在本机重新确认端口，按每个控制器最多 15 个端口完成映射，再关闭 `XhciPortLimit`。
- `itlwm` 不是原生 AirPort 驱动，通常需要 HeliPort；隔空投送、接力等连续互通功能不能按原生 Broadcom 体验预期。
- `CPUFriendDataProvider`、`UTBMap`、SSDT 和设备路径均针对当前硬件，换主板、CPU 或 USB 布局后必须重新制作。
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
- [AppleALC](https://github.com/acidanthera/AppleALC)
- [VirtualSMC](https://github.com/acidanthera/VirtualSMC)
- [IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware)
- [itlwm](https://github.com/OpenIntelWireless/itlwm)
- [LucyRTL8125Ethernet](https://github.com/Mieze/LucyRTL8125Ethernet)
- [USBToolBox](https://github.com/USBToolBox/kext)

## 免责声明

Hackintosh 配置与主板 BIOS、固件版本和具体硬件批次强相关。使用前请自行核对并保留恢复手段；本仓库仅作为个人配置记录与参考。
