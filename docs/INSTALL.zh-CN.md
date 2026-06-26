# 安装说明

这份说明面向 Dell OptiPlex 3080 / 同配置 ALC256 机器。目标是复用已经验证的声卡和 Wi-Fi 修复方案。

## 1. 备份当前 EFI

先完整备份自己的 EFI 分区，建议保留一个可以启动的 U 盘 EFI。不要在没有备份的情况下直接覆盖原配置。

## 2. 放入 ACPI 补丁

把本项目文件复制到自己的 EFI：

```text
EFI/OC/ACPI/SSDT-HPET_RTC_TIMR-FIX.aml
```

然后在 `config.plist` 中确认：

```text
ACPI -> Add -> SSDT-HPET_RTC_TIMR-FIX.aml -> Enabled = true
```

如果你用的是本项目的 `config.sample.plist` 做参考，这一项已经写好。注意它只是合并参考片段，不是完整可启动配置。

## 3. 设置声卡 layout-id

确认声卡设备路径：

```text
DeviceProperties -> Add -> PciRoot(0x0)/Pci(0x1F,0x3)
```

关键值：

```text
layout-id = 67
```

已经验证的结果：

- 前置音频正常
- 后置 Line Out 正常

不要继续添加这些试错参数：

```text
alcid=21
alcid=33
alcverbs=1
```

## 4. 确认声卡相关 kext

`Kernel -> Add` 中需要启用：

```text
Lilu.kext
AppleALC.kext
```

macOS 26 / Darwin 25 如系统缺少 AppleHDA，需要自行按合法来源补齐 `AppleHDA.kext`。本项目不会分发 AppleHDA。

## 5. 确认 Wi-Fi 相关 kext

`Kernel -> Add` 中保留并启用：

```text
Lilu.kext
AirportBrcmFixup.kext
IOSkywalkFamily.kext
IO80211FamilyLegacy.kext
IO80211FamilyLegacy.kext/Contents/PlugIns/AirPortBrcmNIC.kext
FakePCIID.kext
```

推荐顺序：

1. `Lilu.kext`
2. `AirportBrcmFixup.kext`
3. 其他常规 kext
4. `IOSkywalkFamily.kext`
5. `IO80211FamilyLegacy.kext`
6. `IO80211FamilyLegacy.kext/Contents/PlugIns/AirPortBrcmNIC.kext`

## 6. 设置 boot-args

保留：

```text
brcmfx-country=#a brcmfxbeta
```

不要加入：

```text
brcmfx-driver=2
brcmfx-delay=9000
brcmfx-aspm
```

## 7. 检查 SMBIOS

公开仓库里的 `config.sample.plist` 不包含 SMBIOS。请在自己的完整 OpenCore 配置里生成并填写：

```text
SERIAL_REPLACE_WITH_YOUR_OWN
MLB_REPLACE_WITH_YOUR_OWN
UUID_REPLACE_WITH_YOUR_OWN
```

请用 GenSMBIOS 或其他工具生成你自己的信息。

## 8. 重启后测试

先测试声音：

```text
系统设置 -> 声音 -> 输出
```

分别测试：

- 前置耳机孔
- 后置 Line Out

再测试 Wi-Fi：

- Wi-Fi 图标是否出现
- 是否能扫描网络
- 是否能连接网络

如果 Wi-Fi 列表没有刷新，优先清理网络偏好缓存并重启，不要先改 kext 或 boot-args。

## 9. 推荐工具

- ProperTree
- OCAT
- OpenCore Configurator
- OpenCore 官方文档

修改完成后，建议用工具重新检查 OpenCore 配置结构。
