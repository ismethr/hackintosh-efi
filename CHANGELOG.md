# Changelog

## 2026-10-03

- OpenCore 更新至 1.0.8 RELEASE，`BOOTX64.efi` 和 UEFI 驱动同步为同版本。
- 移除 AppleALC（macOS 26 已没有 AppleHDA），以及 itlwm、IntelBluetoothFirmware、IntelBTPatcher、BlueToolFixup（macOS 下不使用无线和蓝牙）。启动参数去掉 `alcid=12`，NVRAM Add 去掉两个 Intel 蓝牙变量。
- 新增 CpuTopologyRebuild 2.0.2 和 NVMeFix 1.1.3，并为 RX 5700 XT 注入自定义 PowerPlay 表。
- WhateverGreen 更新至 1.7.1；VirtualSMC、SMCProcessor、SMCSuperIO 更新至 1.3.8。
- USB 重新映射为 15 个端口，关闭 `XhciPortLimit`。
- `LauncherOption = Full`；`csr-active-config = 40000000`，只关闭 NVRAM 保护。
- 关闭 `EnableWriteUnprotector`、`AppleDebug`、`ApplePanic`、`NormalizeHeaders`；`HibernateMode = None`；APFS `MinDate`、`MinVersion` 恢复默认值 0。
- 删除已禁用的样例内核补丁和 Block、Force 条目。
- README 补充主板型号、无线网卡和音频说明，更新组件版本与关键配置。

## 2026-08-26

- 将仓库从旧的 B360M / i5-9400F / RX 580 配置更新为当前 i7-12700 / RX 5700 XT 配置。
- OpenCore 更新至 1.0.7 RELEASE，并使用同版本官方 `BOOTX64.efi`。
- 同步当前 ACPI、Drivers、Kexts、Resources 与 `config.plist`。
- 对 `SystemSerialNumber`、`MLB`、`SystemUUID` 和 `ROM` 做公开发布脱敏。
- 移除旧系统截图、Windows 启动文件和 macOS AppleDouble 元数据。
- 重写 README，补充硬件、组件版本、双系统、隐私与已知维护项说明。
