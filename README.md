# Dell OptiPlex 3080 Hackintosh Audio & Wi-Fi Patch

适用于 Dell OptiPlex 3080 / 同配置 ALC256 机器的 OpenCore 声卡与 Broadcom Wi-Fi 修复补丁。

本项目免费公开给需要的人使用。配置来自实机验证，重点解决：

- ALC256 前置耳机孔、后置 Line Out 无声或不完整的问题
- AppleALC 已加载但系统没有音频输出设备的问题
- macOS 26 / Darwin 25 下 Broadcom Wi-Fi 能显示、扫描并连接的问题

## 已验证环境

| 项目 | 内容 |
| --- | --- |
| 机型 | Dell OptiPlex 3080 / 同配置主机 |
| 声卡 | Realtek ALC256 |
| 系统 | macOS 26 / Darwin 25 系列 |
| 引导 | OpenCore |
| 声卡状态 | 前置音频正常，后置 Line Out 正常 |
| Wi-Fi 状态 | 可以显示 Wi-Fi、扫描网络并连接 |

## 文件说明

```text
EFI/OC/ACPI/SSDT-HPET_RTC_TIMR-FIX.aml
EFI/OC/config.sample.plist
docs/INSTALL.zh-CN.md
records/verified-fix-notes.zh-CN.txt
```

- `SSDT-HPET_RTC_TIMR-FIX.aml`：HPET / RTC / TIMR IRQ 修复补丁。
- `config.sample.plist`：OpenCore 合并参考片段，不是完整可启动配置。
- `docs/INSTALL.zh-CN.md`：安装和核对步骤。
- `records/verified-fix-notes.zh-CN.txt`：实机验证记录，保留完整排查结论。

## 核心结论

### 声卡

- `EFI/OC/ACPI/SSDT-HPET_RTC_TIMR-FIX.aml` 必须存在，并在 `ACPI -> Add` 中启用。
- `DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x1F,0x3)` 使用 `layout-id = 67`。
- `Kernel -> Add` 中启用 `AppleALC.kext`。
- `boot-args` 不再写 `alcid=21`、`alcid=33`、`alcverbs=1`。
- macOS 26 / Darwin 25 如缺少 AppleHDA，需要自行按合法来源补齐 `AppleHDA.kext`。

### Wi-Fi

Broadcom Wi-Fi 在 macOS 26 / Darwin 25 下需要 Legacy Wi-Fi 组件：

- `Lilu.kext`
- `AirportBrcmFixup.kext`
- `IOSkywalkFamily.kext`
- `IO80211FamilyLegacy.kext`
- `IO80211FamilyLegacy.kext/Contents/PlugIns/AirPortBrcmNIC.kext`
- `FakePCIID.kext`

`boot-args` 保留：

```text
brcmfx-country=#a brcmfxbeta
```

不要再加入：

```text
brcmfx-driver=2
brcmfx-delay=9000
brcmfx-aspm
```

## 使用方式

请先阅读 [安装说明](docs/INSTALL.zh-CN.md)。

简单流程：

1. 备份自己的 EFI。
2. 把 `EFI/OC/ACPI/SSDT-HPET_RTC_TIMR-FIX.aml` 放入自己的 `EFI/OC/ACPI/`。
3. 参考 `EFI/OC/config.sample.plist` 合并 ACPI、DeviceProperties、Kernel、NVRAM 中的关键项。
4. 使用 ProperTree、OCAT 或 OpenCore Configurator 检查配置。
5. 重启后分别测试前置耳机孔、后置 Line Out 和 Wi-Fi。

## 重要提醒

- `config.sample.plist` 是合并参考片段，不要直接覆盖自己的完整配置。
- 请务必生成自己的 SMBIOS 信息，不要使用别人的序列号、UUID、MLB。
- 本项目不包含 AppleHDA.kext、AppleALC.kext、Lilu.kext 等 kext 文件。请从合法来源获取。
- 黑苹果配置与 BIOS、硬件批次、OpenCore 版本有关。使用前请备份 EFI，方便回退。

## 免费使用

本项目按 MIT License 免费公开。你可以复制、修改、分享，但请自行承担使用风险。
