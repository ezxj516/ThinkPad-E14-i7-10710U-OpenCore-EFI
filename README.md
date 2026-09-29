# Lenovo ThinkPad E14 (i7-10710U) Hackintosh EFI — macOS Sequoia 15.8 · OpenCore 1.0.7 · DW1560

> **联想 ThinkPad E14（Comet Lake）黑苹果 EFI 分享**
> ✅ Wi-Fi / 蓝牙已通过更换 **DW1560** 网卡完整驱动
> ❌ **睡眠唤醒失败**（见下方"已知问题"）
> 已在 macOS 15.8 Sequoia (24H23) 上稳定日常使用

---

## 电脑配置

| 项目 | 型号 | 状态 |
|---|---|---|
| 机型 | 联想 ThinkPad E14（Comet Lake 平台） | — |
| CPU | Intel Core i7-10710U（6 核 12 线程，15W） | ✅ 变频正常（睿频 2.7GHz+，15W 功耗墙行为正常） |
| 核显 | Intel UHD Graphics (CFL, 0x3E9B) | ✅ Metal 3 硬件加速 |
| 独显 | AMD Polaris 12 (0x6987) | ⛔ 无驱动，已用 `-wegnoegpu` 屏蔽省电 |
| 内存 | 32GB DDR4 | ✅ About 正确识别 |
| 硬盘 | KIOXIA 1TB NVMe | ✅ 原生 NVMe + TRIM |
| **无线网卡** | **Dell DW1560（BCM94352Z，BCM4352 + BCM20702）** | ✅ **原机 AX210 已换为此卡** |
| 声卡 | Conexant CX8070/CX11880 (layout-id 15) | ✅ 扬声器 + 内置麦克风 |
| 有线 | Realtek RTL8168/8111 | ✅ |
| 键盘/触控板 | PS/2 | ✅ VoodooPS2 |
| 电池/亮度/摄像头/USB | EC / PNLF / USB 映射 | ✅ |

> 📝 **CPU 型号备注**：E14 同机型存在不同 CPU 配置（i5-10210U / i7-10510U / i7-10710U 等）。本机 CPUID 实测为 6 核 12 线程、最大睿频 4.7GHz、基频 1.1GHz，即 i7-10710U。本 EFI 的电源管理对所有 Comet Lake U 系列均适用。

**系统版本**：macOS Sequoia 15.8 (24H23)
**OpenCore**：1.0.7（图形界面 OpenCanopy 已启用）
**SMBIOS**：MacBookPro16,3（⚠️ 分享版已清空序列号，使用前必须自己生成，见下文）

---

## ✅ 正常工作

- 核显硬件加速（Metal 3）、亮度调节（`-igfxblt -igfxbls` 背光修复）
- **Wi-Fi**：原生博通驱动 `AirPort_BrcmNIC`，5GHz 802.11ac 80MHz，系统原生 Wi-Fi 菜单
- **蓝牙**：原生 `bluetoothd`，蓝牙耳机实测配对/连接/自动重连正常
- 声音（输出 + 内置麦克风）、电池状态、键盘/触控板、USB（已做端口映射）
- NVMe TRIM、CPU 原生电源管理（XCPM）、iCloud / App Store 登录
- OpenCanopy 图形启动选单，与 Windows 双系统并存

## ❌ 已知问题

> ### ⚠️ 睡眠唤醒失败
> **本机睡眠（苹果菜单→睡眠、合盖、自动睡眠）后无法唤醒，只能长按电源键强制断电。**
> 已尝试 `-igfxbls`、GPRW 相关思路等均未解决，属于该机型的已知顽疾。
> **EFI 中已把系统配置为永不自动睡眠，但仍请务必：不要手动点"睡眠"，不要不合盖直接关机盖！**
> （合盖会触发睡眠 → 卡死 → 电池耗干。hibernatemode 已设为 0。）

- AMD 独显无 macOS 驱动（已屏蔽，不影响使用，内屏走核显）
- AirDrop / 接力：`awdl0` 接口正常、前置条件齐备，理论上可用（未做完整实测）
- iMessage / FaceTime：配置层面已就绪（en0 为 built-in、ROM 已对齐），未实测
- Wi-Fi 接口编号为 `en3`（`en0` 是有线网卡），纯编号问题不影响功能

---

## 🔁 网卡更换说明：Intel AX210 → Dell DW1560（重点）

原机自带 **Intel AX210**，实测在 macOS Sequoia 上：Wi-Fi 只能用 itlwm + HeliPort（无原生菜单、无 AirDrop），**蓝牙完全无法驱动**（IntelBluetoothFirmware 2.4.0 + IntelBTPatcher + BlueToolFixup 全上后 bluetoothd 仍以 STATUS 718 无限崩溃，A/B 对照确认）。

**更换为 DW1560（BCM94352Z，M.2 2230）后 Wi-Fi + 蓝牙全部原生驱动**，且**实测不需要 OCLP root patch**（系统卷保持封印）。

### DW1560 所需的完整配置（都在本 EFI 里）

**Kexts（按此清单勾选）：**

| Kext | 用途 |
|---|---|
| `IOSkywalkFamily.kext` | Sonoma+ 恢复老 Wi-Fi 栈（MinKernel 23.0.0） |
| `IO80211FamilyLegacy.kext` + 插件 `AirPortBrcmNIC.kext` | 博通 Wi-Fi 驱动本体（插件必须启用！） |
| `AirportBrcmFixup.kext` + 插件 `AirPortBrcmNIC_Injector.kext` | 第三方博通支持（插件必须启用！） |
| `AMFIPass.kext` | 允许上述注入 |
| `BrcmPatchRAM3.kext` + `BrcmFirmwareData.kext` | 蓝牙固件上传 |
| `BlueToolFixup.kext` | Monterey+ 蓝牙栈修复（MinKernel 21.0.0） |

**Kernel → Block：**

| Identifier | 说明 |
|---|---|
| `com.apple.iokit.IOSkywalkFamily` | 屏蔽系统新版 Skywalk（MinKernel 23.0.0，必须） |

**NVRAM（7C436110-AB2A-4BBB-A880-FE41995C9F82）：**

| 变量 | 值 | 说明 |
|---|---|---|
| `csr-active-config` | `03080000` | SIP 部分关闭（本方案必需） |
| `bluetoothInternalControllerInfo` | **14 字节零** `00000000 00000000 00000000 0000` | ⚠️ 必须是 14 字节，不是 16 |
| `bluetoothExternalDongleFailed` | `00`（1 字节） | 配套项 |

**boot-args（与本卡相关部分）：**

```
-btlfxboardid      ← BrcmPatchRAM 2.7.0+ 在 macOS 14+ 上博通蓝牙必需
-btlfxbeta -btlfxallowanyaddr   ← BlueToolFixup 配套
brcmfx-country=US  ← 国家码。注意：设成 CN 会导致 5GHz 只剩 149-165 信道，
                      若你的路由器 5GHz 用 36-64 信道会掉到 2.4GHz，按需修改
```

> 💡 本机体验：CN 域下 5GHz 只剩信道 149-165，路由器在 44 信道会连不上 5G，所以默认 US。

---

## BIOS 设置建议

以下 quirk 已开启，所以**不需要**进 BIOS 改对应选项：

| quirk | 免去的 BIOS 修改 |
|---|---|
| `AppleXcpmCfgLock` | 不需要解锁 CFG Lock |
| `DisableIoMapper` | 不需要关 VT-d |
| `framebuffer-stolenmem/fbmem` 注入 | 不需要改 DVMT Pre-Allocated |

仍建议：**Secure Boot 关闭**、SATA 模式 AHCI（本机默认即满足）。

---

## 使用方法

1. **生成自己的 SMBIOS（必做！）**：用 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) 选 `MacBookPro16,3` 生成 Serial / MLB / SmUUID，ROM 填你的有线网卡 MAC（6 字节），填入 `EFI/OC/config.plist → PlatformInfo → Generic`。分享版这四项是空的。
2. 把 `EFI` 文件夹整个放到目标盘的 EFI 分区（替换或新建 `EFI` 目录）。
3. 开机在 OpenCore 选单里执行一次 **Reset NVRAM**（已内置 `ResetNvramEntry.efi`），再选 macOS 启动。
4. 若换卡前装过 itlwm/HeliPort/Intel 蓝牙相关软件，建议删除避免冲突。

**核显背光说明**：`-igfxblt` / `-igfxbls` 是 WhateverGreen 的 CFL 平台背光寄存器修复（macOS 13.4+ 必需，详见 WEG 手册），**不要删**。

**系统更新注意**：大版本更新前先备份整个 EFI；更新后如 Wi-Fi/蓝牙异常，优先检查本页 kext 是否有新版本。

---

## EFI 结构

```
EFI/OC/
├── ACPI/        SSDT-ALS0/MCHC/PLUG/PNLF/RTCAWAC/SBUS/USB-Reset/USBX/XOSI
├── Drivers/     HfsPlus · OpenCanopy · OpenRuntime · ResetNvramEntry
├── Kexts/       27 个（版本见下）
├── Tools/       OpenShell
└── config.plist
```

| 核心 Kext | 版本 | | 核心细节 | |
|---|---|---|---|---|
| Lilu | 1.7.2 | | VirtualSMC 系列 | 1.3.7 |
| WhateverGreen | 1.7.0 | | AppleALC | 1.9.7 |
| AirportBrcmFixup | 2.2.1 | | BrcmPatchRAM3 / BrcmFirmwareData | 2.7.2 |
| BlueToolFixup | 2.7.2 | | IOSkywalkFamily | 1.0 |
| IO80211FamilyLegacy | 12.0 | | AMFIPass | 1.4.1 |
| VoodooPS2Controller | 2.3.7 | | USBToolBox / UTBMap | 1.2.0 / 1.1 |
| RealtekRTL8111 | 3.0.0 | | NVMeFix | 1.1.3 |
| RestrictEvents | 1.1.6 | | ECEnabler | 1.0.6 |

---

## 致谢

- [Acidanthera](https://github.com/acidanthera)（OpenCore / Lilu / WEG / AppleALC / BrcmPatchRAM / VirtualSMC / VoodooPS2）
- [Dortania](https://github.com/dortania/OpenCore-Legacy-Patcher)（AMFIPass、legacy Wi-Fi kext 与思路）
- [5T33Z0/OCLP4Hackintosh — WiFi_Sonoma 指南](https://github.com/5T33Z0/OCLP4Hackintosh/blob/main/Enable_Features/WiFi_Sonoma.md)（DW1560 配置项的主要依据）
- [OpenIntelWireless](https://openintelwireless.github.io)（itlwm/HeliPort，AX210 时代曾用）
- 原始 EFI 底子的作者（U 盘装机版）

## 免责声明

仅供学习交流，请自行承担使用风险。请勿用于商业用途。使用前请阅读 [Apple EULA 与 Hackintosh 相关法律讨论](https://dortania.github.io/OpenCore-Install-Guide/)。

---

## 关键词 / Keywords

`hackintosh` `黑苹果` `黑果装机` `OpenCore` `OpenCore EFI` `hackintosh efi` `Lenovo ThinkPad E14` `ThinkPad E14 hackintosh` `E14 Gen1` `i7-10710U` `i5-10210U` `Comet Lake` `macOS Sequoia` `macOS 15.8` `Sonoma` `DW1560` `DW1560 hackintosh` `BCM94352Z` `BCM4352` `BCM20702` `Broadcom WiFi` `博通网卡` `换网卡` `睡眠唤醒失败` `sleep wake not working` `AirPortBrcmNIC` `BrcmPatchRAM` `BlueToolFixup`
