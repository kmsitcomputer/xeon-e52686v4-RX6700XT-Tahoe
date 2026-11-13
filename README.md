# Xeon E5-2686 v4 · RX 6700 XT · macOS Tahoe — OpenCore EFI

OpenCore EFI for a **X99 / Xeon E5-2686 v4 + AMD Radeon RX 6700 XT** Hackintosh running **macOS Tahoe (26)**.

> **⚠️ IMPORTANT — Serials are sanitized!**
> `SystemSerialNumber`, `MLB`, `SystemUUID` and `ROM` in `EFI/OC/config.plist` are placeholders (`XXXX…` / `AA AA AA`).
> You **MUST** generate your own values before using this EFI. See [Before you boot](#before-you-boot).

---

## Update log

| Date | Change |
| --- | --- |
| 2026-11-10 | Switched Wi-Fi/BT to **Intel AX210**: added `AirportItlwm` (Tahoe build), `IntelBluetoothFirmware` + `IntelBTPatcher` (lshbluesky fork), removed Realtek Bluetooth firmware. Updated `config.plist` (added `-btlfxallowanyaddr` boot-arg) and refreshed `UTBMap.kext` USB port map. |
| 2026-11-13 | Added `shda-lang=id` boot-arg (SimpleHDA audio language). |

---

## Hardware

| Component | Details |
| --- | --- |
| CPU | Intel Xeon E5-2686 v4 (Broadwell-EP, 18C / 36T, LGA2011-3) |
| Motherboard | X99 (Chinese LGA2011-3 board) |
| GPU | AMD Radeon RX 6700 XT (Navi 22, RDNA 2) |
| Audio | Realtek onboard (ALC) |
| Ethernet | Realtek RTL8111 |
| Wi-Fi | Intel AX210 (Tahoe build of AirportItlwm) |
| Bluetooth | Intel AX210 (IntelBluetoothFirmware + IntelBTPatcher) |
| SMBIOS | `MacPro7,1` (vendor spoofed) |

## Software

| Item | Details |
| --- | --- |
| OpenCore | `DBG-109-2026-10-31` (debug build) |
| macOS | Tahoe (26) — `boot.efi 583~6005` |
| boot-args | `debug=0x100 keepsyms=1 npci=0x2000 -btlfxallowanyaddr shda-lang=id` |
| SIP | Enabled (`csr-active-config = 0`) |
| Secure Boot | `SecureBootModel: Disabled` |
| NVRAM | Emulated (`OpenVariableRuntimeDxe` — X99 legacy NVRAM) |

## ACPI

| SSDT | Purpose |
| --- | --- |
| `SSDT-EC` | Fake embedded controller |
| `SSDT-PLUG` | CPU power management (X99) |
| `SSDT-RTC0-RANGE` | Fix RTC range on X99 |
| `SSDT-SBUS` | SMBus device |
| `SSDT-UNC` | Uncore device for X99 |
| `SSDT-USBX` | USB power properties |

## Drivers

`apfs_aligned.efi`, `HfsPlus.efi`, `OpenCanopy.efi`, `OpenRuntime.efi`, `OpenVariableRuntimeDxe.efi`, `ResetNvramEntry.efi`

## Kexts

| Kext | Version | Purpose |
| --- | --- | --- |
| Lilu | 1.7.3 | Patch engine |
| VirtualSMC | 1.3.9 | Fake SMC |
| SMCProcessor | 1.3.9 | CPU sensors |
| SMCSuperIO | 1.3.9 | Fan sensors |
| NootRX | 1.0.0 | RX 6700 XT (RDNA 2) graphics (min kernel 20.5.0) |
| AppleALC | 1.9.9 | Audio |
| SimpleHDA | 1.0.1 | Audio (Realtek fallback) |
| CpuTscSync | 1.1.3 | TSC sync on X99 |
| NVMeFix | 1.1.4 | NVMe power management |
| RealtekRTL8111 | 3.0.22 | Ethernet |
| AirportItlwm | 2.4.0 (Tahoe) | Wi-Fi Intel AX210 (laobamac build, min kernel 25.0.0) |
| IntelBluetoothFirmware | 2.5.3 (lshbluesky fork) | Bluetooth AX210 firmware |
| IntelBTPatcher | 2.5.3 (lshbluesky fork) | Bluetooth AX210 patcher |
| BlueToolFixup | 2.7.3 | Bluetooth on Monterey+ |
| RestrictEvents | 1.1.7 | Patch RAM warnings / events |
| USBToolBox | 1.2.0 | USB mapping tool |
| UTBMap | 2.0 | USB port map (XHCI 12 ports + EHC1 + EHC2) |
| XHCI-unsupported | 0.9.2 | Unsupported USB controllers |

---

## Before you boot

1. **Generate your own SMBIOS serials** — run [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) and pick `MacPro7,1`, then replace the placeholders in `EFI/OC/config.plist` → `PlatformInfo → Generic`:

   | Key | Replace `XXXX…` with |
   | --- | --- |
   | `SystemSerialNumber` | Serial from GenSMBIOS (12 chars) |
   | `MLB` | MLB from GenSMBIOS (17 chars) |
   | `SystemUUID` | SmUUID from GenSMBIOS |
   | `ROM` | Your NIC MAC address or any random 6 bytes |

2. **Check the USB map** — `UTBMap.kext` is mapped for *this* machine's ports. If USB is broken on your board, remap with [USBToolBox](https://github.com/USBToolBox/tool).

3. **GPU** — this EFI expects an RDNA 2 card (6000 series) via NootRX. Other GPUs need different kexts (e.g. whatevergreen/NVIDIA is unsupported).

## Installation

```bash
# 1. Mount your ESP
diskutil mount diskXs1

# 2. Copy the EFI folder to the ESP root
cp -R EFI /Volumes/ESP/

# 3. Unmount
diskutil unmount diskXs1
```

Then reset NVRAM (`ResetNvramEntry` is available in the boot picker, or press `Cmd+Opt+P+R` / use `Reset NVRAM` in OpenCanopy) and boot.

> **Note:** the `NVRAM` folder is intentionally not included — it is created at runtime by the emulated NVRAM driver on first boot.

## Credits

- [Acidanthera](https://github.com/Acidanthera) — OpenCore, Lilu, VirtualSMC, AppleALC, NootRX, RestrictEvents, CpuTscSync, NVMeFix
- [Mieze](https://github.com/Mieze) — RealtekRTL8111
- [USBToolBox](https://github.com/USBToolBox) — USB mapping
- [laobamac](https://github.com/laobamac) — AirportItlwm macOS Tahoe build (Intel Wi-Fi)
- [lshbluesky](https://github.com/lshbluesky) — IntelBluetoothFirmware/IntelBTPatcher Tahoe fork
- [corpnewt](https://github.com/corpnewt) — GenSMBIOS
- [Dortania](https://dortania.github.io) — OpenCore Install Guide
