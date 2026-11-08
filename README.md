<div align="center">

# ⚡ HP VICTUS 15-fa0xxx OpenCore EFI
###  High-Performance macOS Hackintosh Configuration for Intel 12th Gen Alder Lake-H

[![OpenCore](https://img.shields.io/badge/OpenCore-v1.0.x-00599C?style=for-the-badge&logo=apple&logoColor=white)](https://github.com/acidanthera/OpenCorePkg)
[![macOS Support](https://img.shields.io/badge/macOS-Monterey%20|%20Ventura%20|%20Sonoma%20|%20Sequoia-black?style=for-the-badge&logo=apple&logoColor=white)](https://www.apple.com/macos/)
[![CPU](https://img.shields.io/badge/CPU-Intel%20Core%20i5--12500H-0071C5?style=for-the-badge&logo=intel&logoColor=white)](https://ark.intel.com/)
[![SMBIOS](https://img.shields.io/badge/SMBIOS-MacBookPro16%2C4-FF2D20?style=for-the-badge&logo=apple&logoColor=white)](https://everymac.com/)
[![Status](https://img.shields.io/badge/Build%20Status-Experimental%20Accel-orange?style=for-the-badge&logo=checkmarx&logoColor=white)](#-feature-compatibility-matrix)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

<br/>

<p align="center">
  <b>A comprehensive, state-of-the-art OpenCore EFI configuration tailored for the HP Victus 15 Gaming Laptop (15-fa0xxx series).</b><br/>
  Engineered specifically to support Intel's 12th Generation Alder Lake hybrid architecture (Performance + Efficient Cores), resolve HP's notorious RTC Power Loss 005 post errors, drive the RTX 3050 through the experimental WhateverXe 0.7.0 acceleration stack, run the stock MediaTek MT7921 Wi-Fi via an experimental community driver, and provide surgical ACPI hotpatches.
</p>

---

</div>

## 📑 Table of Contents
- [📌 System Overview](#-system-overview)
- [💻 Hardware Specifications](#-hardware-specifications)
- [📊 Feature Compatibility Matrix](#-feature-compatibility-matrix)
  - [🟢 Fully Working Features](#-fully-working-features)
  - [🔴 Unsupported Features & In-Depth Technical Rationale](#-unsupported-features--in-depth-technical-rationale)
  - [🟡 Partial / Experimental Features](#-partial--experimental-features)
- [⚙️ BIOS / UEFI Configuration](#️-bios--uefi-configuration)
- [🧩 ACPI Architecture & DSDT Patches](#-acpi-architecture--dsdt-patches)
- [📦 Kexts Inventory & Injected Drivers](#-kexts-inventory--injected-drivers)
- [🚀 Boot Arguments & Kernel Flags](#-boot-arguments--kernel-flags)
- [🛠️ Installation Guide & SMBIOS Setup](#️-installation-guide--smbios-setup)
- [💡 Troubleshooting & Frequently Asked Questions](#-troubleshooting--frequently-asked-questions)
- [🤝 Credits & Acknowledgements](#-credits--acknowledgements)
- [⚖️ License & Disclaimer](#️-license--disclaimer)

---

## 📌 System Overview

This repository provides an enterprise-grade, battle-tested OpenCore EFI tree designed for running macOS on the **HP Victus 15-fa0xxx** platform. It handles Alder Lake-H hybrid CPU core scheduling natively, boots the stock MediaTek MT7921 radios on the experimental `AirportMtkWlan` driver pair, and leans on the bleeding-edge `WhateverXe` 0.7.0 stack to push low-level NVIDIA components toward acceleration instead of powering the RTX 3050 down. HP-specific firmware quirks (RTC 005) remain hotpatched.

| System Attribute | Configuration / Hardware Profile |
| :--- | :--- |
| **Target Machine** | HP Victus Gaming Laptop 15-fa0xxx (Motherboard HP 8A25) |
| **Platform Architecture** | Intel Alder Lake-H (Intel 600 Series Mobile Chipset) |
| **Processor (CPU)** | 12th Gen Intel Core i5-12500H (12 Cores / 16 Threads: 4 P-Cores + 8 E-Cores, up to 4.50 GHz) |
| **Integrated GPU (iGPU)** | Intel Iris Xe Graphics (PCI ID `8086:46A6`, Spoofed to `0x46D2` via WhateverXe) |
| **Discrete GPU (dGPU)** | NVIDIA GeForce RTX 3050 Laptop GPU 4GB GDDR6 (Ampere, kept powered for experimental `WhateverXe 0.7.0` acceleration) |
| **Bootloader** | OpenCore 1.0.x (UEFI Mode) |
| **Target SMBIOS** | `MacBookPro16,4` (or `MacBookPro16,2`) |
| **Supported macOS Versions** | macOS Monterey (12.x) • macOS Ventura (13.x) • macOS Sonoma (14.x) • macOS Sequoia (15.x) |

### 🆕 Latest EFI Changes (Nov 2026)

| Area | Before | Now |
| :--- | :--- | :--- |
| **dGPU RTX 3050** | Powered down via `SSDT-Disable_GPU_PEG1.aml` | Kept powered; experimental acceleration via `WhateverXe 0.7.0` + NV boot-args |
| **Wi-Fi MT7921** | Slot disabled via `SSDT-Disable_Network_RP07.aml` | Runs via experimental `AirportMtkWlan.kext` (freeze reports under load) |
| **Bluetooth MT7921** | Disabled | Experimental `SimpleMtkBtFw.kext` firmware loader |
| **Audio** | `AppleALC layout-id: 12` | `AppleALC layout-id: 11` |
| **iGPU DeviceProperties** | `framebuffer-patch-enable` + `framebuffer-stolenmem` | Stripped — only `device-id` injection remains |
| **Kernel panic capture** | Off (`ApplePanic: false`) | On (`ApplePanic: true`) — panics are saved to disk |
| **SIP / AMFI** | `csr-active-config 0x03` | Raised `csr-active-config` + `amfi_get_out_of_my_way=0x1` for OCLP-style root patches |

---

## 💻 Hardware Specifications

A detailed breakdown of all physical hardware components installed in the **HP Victus 15-fa0xxx**, along with their respective device IDs, kernel drivers, and operational status under macOS:

| Component | Hardware Details | Device / Vendor ID | Handler / Driver / Kext | Status |
| :--- | :--- | :--- | :--- | :---: |
| **Laptop Model** | HP Victus Gaming Laptop 15-fa0xxx | System Board ID: `8A25` | Custom ACPI Tables / DSDT Hotpatches | `Native` |
| **Processor (CPU)** | Intel Core i5-12500H @ 2.50GHz (Turbo 4.50GHz, 18MB Cache) | `0x46A6` (Alder Lake-H) | `CpuTopologyRebuild.kext` + `SSDT-PLUG-ALT.aml` | 🟢 Normal |
| **Motherboard Chipset** | Intel Alder Lake PCH (Intel 600 Series Chipset) | `0x51A4` / Intel 600 Series | `SSDT-MCHC.aml` + `SSDT-SBUS.aml` | 🟢 Normal |
| **Integrated GPU (iGPU)** | Intel Iris Xe Graphics (80 Execution Units) | `8086:46A6` (Spoofed `0x46D2`) | `WhateverXe.kext 0.7.0` + DeviceProperties + NV boot-args | 🟡 Experimental |
| **Discrete GPU (dGPU)** | NVIDIA GeForce RTX 3050 Mobile 4GB GDDR6 (GA107M) | `10DE:25A0` (Ampere Architecture) | `WhateverXe.kext 0.7.0` NVIDIA path (experimental) | 🟡 Experimental |
| **Display Panel** | 15.6" FHD (1920x1080) 144Hz IPS Anti-Glare eDP | Internal eDP Connection | `SSDT-PNLF.aml` + `BrightnessKeys.kext` | 🟢 Normal |
| **Memory (RAM)** | 8GB / 16GB / 32GB DDR4 3200MHz Dual-Channel SODIMM | DDR4 Non-ECC Memory | Native Apple Memory Controller Management | 🟢 Normal |
| **Storage (NVMe SSD)** | PCIe 4.0 / PCIe 3.0 x4 M.2 2280 NVMe Solid State Drive | Standard NVMe PCI Controller | `NVMeFix.kext` (Enables APFS Trim & Autonomous ASPM) | 🟢 Normal |
| **Audio Codec** | Realtek High Definition Audio (ALC245 / ALC287) | `10EC:0245` / `10EC:0287` | `AppleALC.kext` (`layout-id: 11`) | 🟢 Normal |
| **Ethernet (LAN)** | Realtek RTL8111 / RTL8168 PCI-E Gigabit Ethernet | `10EC:8168` | `RealtekRTL8111.kext` | 🟢 Normal |
| **Wireless LAN (Wi-Fi)**| MediaTek MT7921 Wi-Fi 6 (M.2, stock card) | `14C3:7961` | `AirportMtkWlan.kext` (experimental community driver) | 🟡 Unstable |
| **Bluetooth** | MediaTek Bluetooth 5.2 (MT7921 combo) | Internal USB Interface | `SimpleMtkBtFw.kext` (experimental firmware loader) | 🟡 Unstable |
| **Touchpad** | HP Precision I2C Multi-Touch Trackpad | `ELAN` / `SYNA` I2C HID | `VoodooI2C.kext` + `VoodooI2CHID.kext` (Force Polling) | 🟢 Normal |
| **Keyboard** | HP Standard Backlit PS/2 Keyboard | Internal PS/2 Controller | `VoodooPS2Controller.kext` | 🟢 Normal |
| **Webcam** | HP Wide Vision 720p HD Camera (Temporal Noise Reduction) | USB UVC Internal (`05C8:...`) | Native Apple USB Camera Class Driver | 🟢 Normal |
| **Battery & Power** | HP 3-cell, 52.5 Wh / 70 Wh Li-ion Polymer Battery | HP Embedded Controller (EC) | `VirtualSMC.kext` + `SMCBatteryManager` + `ECEnabler` | 🟢 Normal |
| **USB Connectivity** | 1x USB Type-C 5Gbps (DP 1.4), 2x USB Type-A 5Gbps | Intel xHCI Controller | `USBToolBox.kext` + `UTBDefault.kext` (Mapped) | 🟢 Normal |
| **SD Card Reader** | Realtek PCIe SD Card Reader | `10EC:522A` | Unmapped Realtek RTSX Controller | 🔴 Unsupported |

---

## 📊 Feature Compatibility Matrix

### 🟢 Fully Working Features

| Category | Component / Subsystem | Status | Technical Details & Operational Behavior |
| :--- | :--- | :---: | :--- |
| **CPU Power Management** | Intel Core i5-12500H (P & E Cores) | 🟢 WORKING | Full power scaling and C-states handled via `SSDT-PLUG-ALT.aml` and `CpuTopologyRebuild.kext`. P-cores and E-cores dynamically share load; Turbo Boost accelerates seamlessly up to 4.5GHz. |
| **Audio Output** | Internal Speakers & 3.5mm Headphone Jack | 🟢 WORKING | Driven by `AppleALC.kext` using `layout-id: 11`. Crystal clear sound, zero distortion, with instantaneous automatic switching when plugging in headphones. |
| **Audio Input** | Internal Dual-Array Digital Microphones | 🟢 WORKING | Flawless audio capture in FaceTime, Zoom, Discord, and native Voice Memos. |
| **Display & Backlight** | Internal Screen & Smooth Backlight Slider | 🟢 WORKING | Native 1080p eDP display output with smooth brightness transitions controlled via the macOS Control Center slider and function keys (`F2`/`F3`). |
| **Keyboard & Backlight** | Internal Keyboard & LED Backlight | 🟢 WORKING | Responsive typing with zero latency, full multimedia/volume shortcut keys active, and backlit illumination operating correctly. |
| **Trackpad / Gestures** | I2C Multi-Touch Precision Gestures | 🟢 WORKING | 1, 2, 3, and 4-finger gestures (pinch-to-zoom, desktop swipe, Mission Control, natural scrolling) operate fluidly using `-vi2c-force-polling`. |
| **Battery Telemetry** | Battery Percentage, AC Charging & Health | 🟢 WORKING | 16-bit HP Embedded Controller registers read via `ECEnabler.kext` and reported by `SMCBatteryManager.kext`. Cycle count, charge status, and remaining time display accurately. |
| **Wired Network** | Realtek Gigabit Ethernet RJ-45 Port | 🟢 WORKING | Full 10/100/1000 Mbps line speeds out of the box via `RealtekRTL8111.kext`. Reliable connection for heavy file transfers. |
| **USB Bus Architecture** | USB 3.2 Gen 1 Type-A & Type-C Ports | 🟢 WORKING | Fully mapped within the macOS 15-port limit via `USBToolBox.kext` and `UTBDefault.kext`. Proper 500mA/900mA/1500mA power injection through `SSDT-USBX.aml`. |
| **Internal HD Webcam** | HP Wide Vision 720p HD Camera | 🟢 WORKING | Standard USB UVC webcam recognized natively by Apple's camera stack without third-party patches. Fully supported in Photo Booth and video conferences. |
| **Storage Architecture** | M.2 NVMe SSD Read/Write & APFS Boot | 🟢 WORKING | High-speed PCIe Gen4/Gen3 storage operation with active Autonomous Power State Management (ASPM) and background APFS TRIM enabled by `NVMeFix.kext`. |
| **System Sleep & Wake** | S3 / Modern Standby Emulation | 🟢 WORKING | Enters deep sleep and wakes instantly on lid open or keyboard keystrokes without kernel panics or display blackouts. |
| **Apple Services** | iCloud, App Store, iMessage & FaceTime | 🟢 WORKING | Fully operational provided a unique SMBIOS serial number is generated and the Ethernet interface is registered as `en0` (Built-in). |

---

### 🔴 Unsupported Features & In-Depth Technical Rationale

This section details every hardware component that **cannot operate** in the macOS environment, providing exhaustive technical justifications and recommended workarounds:

| Component / Feature | Operational Status | Technical Root Cause & Recommended Workaround |
| :--- | :---: | :--- |
| **NVIDIA GeForce RTX 3050 Laptop GPU** | 🟡 EXPERIMENTAL | **Root Cause:** Apple deprecated NVIDIA graphics support after macOS High Sierra, and modern Ampere architecture (RTX 30xx) has zero official driver support under Darwin. Furthermore, HP Victus uses NVIDIA Optimus (muxless dynamic switching), an architecture macOS cannot drive natively.<br>**Handling:** The hard power-down (`SSDT-Disable_GPU_PEG1.aml`) has been removed in the Nov 2026 update — the card stays powered and acceleration is attempted with `WhateverXe.kext 0.7.0` plus the NVIDIA-stack boot-args (`nvaccel`, `nvfb`, unlocked AMFI/SIP for OCLP-style root patches). Metal/GL support must be validated app-by-app; if stability regresses, restore a dGPU-disable SSDT or fall back to the Iris Xe iGPU only. |
| **External HDMI Video Output** | 🟡 CONDITIONAL | **Root Cause:** The physical HDMI port on the HP Victus 15 motherboard is hardwired directly to the discrete NVIDIA GeForce RTX 3050 GPU. With the dGPU now kept powered, HDMI can only emit a signal while the patched NVIDIA stack actually drives the card — still staged lacking trial under the experimental acceleration.<br>**Workaround:** Validate after applying the OCLP-style root patches; otherwise use a DisplayLink USB 3.0 / USB-C docking adapter for wired external displays. |
| **Stock MediaTek MT7921 Wi-Fi & BT** | 🟡 UNSTABLE | **Root Cause:** The MT7921 combo card now runs through the experimental community driver `AirportMtkWlan.kext` (+ `SimpleMtkBtFw.kext` for Bluetooth). Association and basic internet work on Sequoia, but the driver is known to crash after long sessions — because `debug=0x100` halts on kernel panic, that crash presents itself as a system "freeze" needing a hard power-off.<br>**Recommendations:**<br>1. *Stability first:* set both kexts `Enabled: false` in `config.plist` and use Ethernet (`en0`) or USB tethering.<br>2. *Clean long-term fix:* swap the M.2 Key-E card for an **Intel AX210** driven by `AirportItlwm.kext` + `IntelBluetoothFirmware.kext` (or a native Broadcom BCM94360NG via OCLP).<br>3. *External alternative:* supported USB Wi-Fi dongle. |
| **Realtek PCIe SD Card Reader** | 🔴 UNSUPPORTED | **Root Cause:** The onboard Realtek RTS522A card reader communicates via the PCIe bus and lacks a maintained, stable macOS driver for modern versions (Sonoma / Sequoia).<br>**Workaround:** Use a generic USB-C / USB-A SD card reader. |
| **AirDrop, Continuity & Universal Control** | 🔴 CONDITIONAL | **Root Cause:** Native Apple wireless continuity features require an Apple-native Broadcom wireless chipset supporting Apple Wireless Direct Link (AWDL) and Bluetooth Low Energy (BLE). Upgrading the internal card to a compatible Broadcom/Intel card will restore partial or full capability. |

---

### 🟡 Partial / Experimental Features

| Subsystem | State | Engineering Explanation |
| :--- | :---: | :--- |
| **Intel Iris Xe Graphics (Alder Lake-H)** | 🟡 Experimental | Intel 12th Gen Alder Lake iGPUs are not natively supported by macOS because Apple transitioned to Apple Silicon prior to their release. This EFI ships the experimental **`WhateverXe.kext` 0.7.0** binary (now including the NVIDIA low-level path) with only a `device-id` injection under `DeviceProperties` — the old `framebuffer-patch-enable` / `framebuffer-stolenmem` properties were removed — and kernel flags `--allow3d --disablefirmware`. The internal display operates in native 1080p, while 3D/Metal paths now share the same NV-acceleration experiment as the RTX 3050. |
| **Touchpad Interrupt Architecture** | 🟡 Polling Mode | HP Victus laptops feature aggressive BIOS interrupt routing for APIC and GPIO controllers. To prevent pointer stutter or lockups upon wake, the touchpad is configured in forced polling mode via the `-vi2c-force-polling` boot argument, guaranteeing butter-smooth input tracking. |
| **MediaTek MT7921 Wi-Fi & Bluetooth** | 🟡 Unstable Driver | Both radios run through the experimental `AirportMtkWlan.kext` + `SimpleMtkBtFw.kext` pair. Connectivity itself works, but extended sessions can panic the kernel (system freeze under `debug=0x100`). Ethernet is kept as the reliable `en0` fallback for services bound to a network identity. |

---

## ⚙️ BIOS / UEFI Configuration

Before attempting to boot into the macOS installer, configure your HP Victus 15 BIOS settings. Access the BIOS setup by powering on the laptop and immediately tapping **`F10`** (or press **`ESC`** and select `F10 BIOS Setup`).

### 🔴 Mandatory Settings to DISABLE
| BIOS Tab / Category | Setting Name | Required Value | Technical Justification |
| :--- | :--- | :---: | :--- |
| **Security** | **Secure Boot** | ❌ **Disabled** | Prevents the HP firmware from refusing execution of OpenCore binaries |
| **Configuration** | **Fast Boot** | ❌ **Disabled** | Ensures full hardware re-enumeration on every reboot |
| **Configuration** | **Intel VMD Technology / Intel RST** | ❌ **Disabled (AHCI)** | **CRITICAL:** macOS cannot detect NVMe drives when Intel VMD RAID controller is engaged |
| **Configuration** | **Wake on LAN / Network Boot** | ❌ **Disabled** | Prevents sleep wake loops caused by Realtek Ethernet sleep states |

### 🟢 Mandatory Settings to ENABLE
| BIOS Tab / Category | Setting Name | Required Value | Technical Justification |
| :--- | :--- | :---: | :--- |
| **Configuration** | **Virtualization Technology (VT-x)** | ✔️ **Enabled** | Required for Darwin kernel hypervisor subsystems and general stability |
| **Configuration** | **VT-d (Direct I/O)** | ✔️ **Enabled** | Supported natively (handled by OpenCore quirk `DisableIoMapper: true`) |
| **Advanced** | **USB Boot / Boot from USB** | ✔️ **Enabled** | Allows the machine to boot from the OpenCore USB recovery media |
| **Advanced** | **UEFI Boot Order** | ✔️ **UEFI First** | Avoids legacy CSM fallback |

> [!IMPORTANT]
> **HP Victus RTC Power Loss (005) Fix:**  
> On HP gaming laptops, rebooting macOS frequently triggers a black screen with the message:  
> `The CMOS checksum is invalid / Real-Time Clock Power Loss (005)`.  
> This EFI incorporates a custom **ACPI RTC 005 Hotpatch** and enables `DisableRtcChecksum: true`, permanently resolving this error without requiring manual BIOS resets.

---

## 🧩 ACPI Architecture & DSDT Patches

All ACPI binaries in this EFI have been compiled with the latest iASL compiler, free of syntax errors or namespace collisions:

| SSDT File Name | Functional Purpose in macOS |
| :--- | :--- |
| [`SSDT-ALS0.aml`](OC/ACPI/SSDT-ALS0.aml) | Injects a dummy Ambient Light Sensor device (`ALS0`) to satisfy Apple's backlight controller |
| [`SSDT-EC.aml`](OC/ACPI/SSDT-EC.aml) | Injects an Apple-compliant fake Embedded Controller (`EC`) device |
| [`SSDT-GPI0.aml`](OC/ACPI/SSDT-GPI0.aml) | Enables the GPIO pin controller for hardware I2C touchpad communication |
| [`SSDT-MCHC.aml`](OC/ACPI/SSDT-MCHC.aml) | Supplies missing Memory Controller Hub Configuration (`MCHC`) devices for Intel 600 Series |
| [`SSDT-PLUG-ALT.aml`](OC/ACPI/SSDT-PLUG-ALT.aml) | Defines CPU Power Management for 12th Gen Alder Lake P-cores and E-cores |
| [`SSDT-PNLF.aml`](OC/ACPI/SSDT-PNLF.aml) | Attaches the `PNLF` backlight device to the internal display for smooth brightness control |
| [`SSDT-RTCAWAC.aml`](OC/ACPI/SSDT-RTCAWAC.aml) | Forces legacy RTC enable while disabling the conflicting modern AWAC hardware clock |
| [`SSDT-SBUS.aml`](OC/ACPI/SSDT-SBUS.aml) | Adds the System Management Bus controller for sensor communication |
| [`SSDT-USB-Reset.aml`](OC/ACPI/SSDT-USB-Reset.aml) | Resets USB companion controllers prior to macOS kernel handover |
| [`SSDT-USBX.aml`](OC/ACPI/SSDT-USBX.aml) | Injects USB power properties, enabling 1500mA fast-charging on compatible ports |
| [`SSDT-XOSI.aml`](OC/ACPI/SSDT-XOSI.aml) | Spoofs Windows OS queries so the HP firmware initializes laptop hardware properly |

### 🛠️ In-Memory ACPI Binary Patches in `config.plist`
* **Fix HP Real-Time Clock Power Loss (005) Post Error:**  
  * `Find`: `RwFwAHAAAQg=` (`47 01 70 00 70 00 01 08`)  
  * `Replace`: `RwFwAHAAAQI=` (`47 01 70 00 70 00 01 02`)  
  * *Purpose:* Eliminates the POST checksum reset error 005 on restart.
* **GPI0 _STA to XSTA Rename:** Allows `SSDT-GPI0.aml` to activate the GPIO controller.
* **PNLF to XNLF Rename:** Prevents duplicate backlight declarations on internal eDP.
* **_OSI to XOSI Rename:** Redirects `_OSI` queries to `SSDT-XOSI.aml`.

---

## 📦 Kexts Inventory & Injected Drivers

All 23 essential kernel extensions loaded in exact execution hierarchy:

| Load Order | Kext Name | Category | Primary Function |
| :---: | :--- | :---: | :--- |
| 1 | **Lilu.kext** | Core Framework | Arbitrary kernel patcher and dependency engine for all extensions |
| 2 | **VirtualSMC.kext** | SMC Emulator | Advanced Apple SMC chip emulation for sensors and system management |
| 3 | **WhateverXe.kext** | Display Driver | Experimental Intel Iris Xe / NVIDIA low-level acceleration driver, v0.7.0 |
| 4 | **AppleALC.kext** | Audio Engine | HD Audio codec injection for Realtek ALC245 / ALC287 (`layout-id: 11`) |
| 5 | **CpuTopologyRebuild.kext** | CPU Scheduler | Optimizes heterogeneous thread scheduling across P-cores and E-cores |
| 6 | **RestrictEvents.kext** | System Utilities | Injects correct CPU branding strings (`revcpuname`) and UI patches |
| 7 | **ECEnabler.kext** | ACPI EC Enabler | Enables reading 16-bit+ Embedded Controller fields on HP machines |
| 8 | **SMCBatteryManager.kext** | Battery Driver | Reads battery percentage, cycle count, and charging health |
| 9 | **SMCProcessor.kext** | CPU Sensor | Telemetry for CPU package power, frequencies, and thermal metrics |
| 10 | **SMCSuperIO.kext** | Fan Sensor | Telemetry for hardware fan speeds |
| 11 | **SMCLightSensor.kext** | Light Sensor | Ambient light sensor dummy integration |
| 12 | **VoodooPS2Controller.kext** | Keyboard Driver | Keyboard driver for the internal HP PS/2 keyboard |
| 13 | **BrightnessKeys.kext** | ACPI Hotkeys | Binds brightness increase/decrease function keys to macOS backlight |
| 14 | **VoodooI2C.kext** | I2C Subsystem | Core controller engine for precision I2C bus devices |
| 15 | **VoodooI2CHID.kext** | Trackpad Engine | Translates multi-touch HID gestures to native macOS events |
| 16 | **RealtekRTL8111.kext** | Ethernet Driver | Full gigabit speed driver for the onboard Realtek RTL8111 NIC |
| 17 | **NVMeFix.kext** | Storage Power | Autonomous power management (ASPM) and APFS TRIM optimizer |
| 18 | **USBToolBox.kext** | USB Framework | Custom USB port injection framework |
| 19 | **UTBDefault.kext** | USB Port Map | Specific physical port mapping map tailored for the HP Victus 15 |
| 20 | **XHCI-unsupported.kext** | USB Controller | Extended chipset support for non-Apple xHCI controllers |
| 21 | **AMFIPass.kext** | Security Shim | Preserves Apple Mobile File Integrity functionality with custom kexts |
| 22 | **AirportMtkWlan.kext** | Wireless Driver | Experimental MediaTek MT7921 Wi-Fi driver (known to panic under load) |
| 23 | **SimpleMtkBtFw.kext** | Bluetooth Driver | MediaTek MT7921 Bluetooth firmware loader |

---

## 🚀 Boot Arguments & Kernel Flags

The boot arguments configured in `NVRAM -> 7C436110-AB2A-4BBB-A880-FE41995C9F82 -> boot-args`:

```bash
debug=0x100 keepsyms=1 cpus=1 -vi2c-force-polling ctrsmt=full --allow3d --disablefirmware nvfb=1 nvaccel=1 nvfbheads=4 -nvkmsnosmooth amfi_get_out_of_my_way=0x1 amfi=0x80 ipc_control_port_options=0 -nvrmnobootscreen -nvrmnoflip gfxrst=4
```

### 🔍 Parameter Breakdown:
* `debug=0x100`: Halts the machine on kernel panic instead of auto-rebooting, preserving error callstacks. Note: this makes a panic look like a "freeze" — the system waits silently for a hard reset instead of restarting.
* `keepsyms=1`: Keeps debug symbols in memory to generate readable backtraces during panics.
* `cpus=1`: Restricts the kernel to a single CPU thread. This is a debugging/bring-up aid used with the OCLP-style acceleration stack and **severely limits multi-core performance** — remove it once the dGPU path is verified.
* `-vi2c-force-polling`: **Crucial for HP Victus.** Forces polling mode for the I2C touchpad, eliminating pointer freezing and GPIO interrupt lockups.
* `ctrsmt=full`: Optimizes simultaneous multithreading allocation inside `CpuTopologyRebuild.kext`.
* `--allow3d`: Experimental WhateverXe switch enabling the 3D accelerator path.
* `--disablefirmware`: Skips firmware loading in the graphics init path to prevent startup hangs.
* `nvfb=1` / `nvaccel=1` / `nvfbheads=4`: Enables the NVIDIA framebuffer + acceleration path within WhateverXe with four display heads.
* `-nvkmsnosmooth` / `-nvrmnobootscreen` / `-nvrmnoflip`: NVIDIA KMS behavior switches used by the patched stack (no smooth mouse, no boot screen takeover, no KMS page flip).
* `gfxrst=4`: Controls graphics-reset behavior of the patched stack.
* `amfi_get_out_of_my_way=0x1` + `amfi=0x80`: Unlocks Apple Mobile File Integrity so unsigned OCLP-style frameworks/kexts load (AMFIPass.kext complements this).
* `ipc_control_port_options=0`: Keeps Electron apps (VS Code, Discord, etc.) functional under the AMFI-unlocked configuration.

---

## 🛠️ Installation Guide & SMBIOS Setup

> [!WARNING]
> **GENERATE YOUR OWN UNIQUE SERIAL NUMBERS!**  
> The `config.plist` includes generic placeholder serials. You **MUST** generate your own distinct serials before logging into your Apple ID or iCloud account.

### Step 1: Generate a Unique SMBIOS
1. Download **[GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)** by CorpNewt.
2. Run `GenSMBIOS.bat` (Windows) or `GenSMBIOS.command` (macOS).
3. Select Option **`1`** to download the latest `MacSerial` binary.
4. Select Option **`2`** and provide the absolute path to your `config.plist`:  
   `...\tools\hp-victus-15-hackintosh\OC\config.plist`
5. Select Option **`3`** and enter the SMBIOS model: **`MacBookPro16,4`**.
6. GenSMBIOS will populate valid unique values for:
   * `SystemProductName`: `MacBookPro16,4`
   * `SystemSerialNumber`: Your randomized serial
   * `MLB`: Your board serial number
   * `SystemUUID`: Your distinct SmUUID

### Step 2: Create the Bootable macOS Installer
1. Download official macOS recovery images (compatible with **Monterey 12**, **Ventura 13**, **Sonoma 14**, or **Sequoia 15**).
2. Create the bootable flash drive using macOS Terminal:
   ```bash
   sudo /Applications/Install\ macOS\ Sequoia.app/Contents/Resources/createinstallmedia --volume /Volumes/MyUSB
   ```
3. Mount the flash drive's EFI partition using MountEFI or ESP Mounter Pro.
4. Copy the entire `EFI` folder (`BOOT` and `OC`) into the root of the EFI partition.

### Step 3: BIOS Setup & Installation
1. Insert the flash drive into a USB Type-A port on the HP Victus.
2. Power on the laptop and tap **`F9`** repeatedly to bring up the **Boot Device Options Menu**.
3. Select your USB drive (UEFI mode).
4. In OpenCore Boot Picker, choose **`Install macOS ... (External)`**.
5. Once macOS Recovery loads, open **Disk Utility** (*View -> Show All Devices*).
6. Format your internal target NVMe drive:
   * **Name**: `Macintosh HD` (or desired name)
   * **Format**: `APFS`
   * **Scheme**: `GUID Partition Map`
7. Exit Disk Utility and proceed with the installation wizard. The laptop will restart 2-3 times during the process.

---

## 💡 Troubleshooting & Frequently Asked Questions

### 1. HP Firmware Error: "Real-Time Clock Power Loss (005)" on Reboot
* **Symptom:** Laptop displays a black screen POST warning regarding CMOS Checksum / RTC power loss after restarting macOS.
* **Cause:** macOS writes an unsupported checksum value into the RTC CMOS memory locations.
* **Resolution:** Ensure the ACPI patch `Fix HP Real-Time Clock Power Loss (005) Post Error` is enabled under `config.plist -> ACPI -> Patch` and that the quirk `DisableRtcChecksum: true` is active. Both are pre-configured in this EFI.

### 2. Trackpad Cursor Stutter or Freeze
* **Resolution:** Ensure the `-vi2c-force-polling` boot argument is present in `NVRAM -> boot-args`. Once booted into macOS, navigate to *System Settings -> Trackpad* and enable "Tap to click".

### 3. Stuck at `[EB|#LOG:EXITBS:START]` during Boot
* **Resolution:** 
  1. Confirm that **Intel VMD / Intel RST** is set to **AHCI** (Disabled) in your laptop BIOS.
  2. Verify that quirks `DevirtualiseMmio`, `SetupVirtualMap`, and `ProvideCurrentCpuInfo` are set to `true`.

### 4. Apple ID / FaceTime / iMessage Sign-In Failures
* **Resolution:** 
  1. Open Terminal and run `ifconfig`. Verify that your Realtek Gigabit Ethernet adapter is listed as interface `en0`.
  2. Generate a fresh serial number using GenSMBIOS under the `MacBookPro16,4` profile.

### 5. Random System Freeze / Hang While Wi-Fi Is On (MT7921)
* **Symptom:** After a long session, the whole machine locks up while the internal MediaTek Wi-Fi is active; only a hard power-off recovers it.
* **Cause:** `AirportMtkWlan.kext` is an experimental driver and can crash the kernel. Because `debug=0x100` halts on panic instead of rebooting, that crash presents itself as a total freeze.
* **Resolution:** Pick one:
  1. Set `AirportMtkWlan.kext` and `SimpleMtkBtFw.kext` to `Enabled: false` in `config.plist -> Kernel -> Add` and use Ethernet / USB tethering.
  2. Swap the M.2 card for an Intel AX210 (`AirportItlwm.kext` + `IntelBluetoothFirmware.kext`).

### 6. Reading Kernel Panic Logs After a Freeze
* `ApplePanic` is now **enabled** (`Misc -> Debug`), so panic reports are no longer lost.
* After rebooting into macOS, open `/Library/Logs/DiagnosticReports/` and look for `Kernel-*.panic` files — they reveal the crashing kext.
* To also see panic text on screen at the moment of the crash, temporarily add `-v` to the boot-args (verbose mode is currently off in this EFI).

---

## 🤝 Credits & Acknowledgements

* **[Acidanthera](https://github.com/acidanthera)** — OpenCore Bootloader, Lilu, AppleALC, VirtualSMC, VoodooPS2, and essential kexts.
* **[b00t0x](https://github.com/b00t0x)** — Developer of the experimental WhateverXe graphics framework.
* **[VoodooI2C Team](https://github.com/VoodooI2C/VoodooI2C)** — Precision I2C touchpad driver architecture.
* **[Dortania](https://dortania.github.io/)** — The gold standard of OpenCore installation guides.
* **[CorpNewt](https://github.com/corpnewt)** — Creator of GenSMBIOS, ProperTree, and USBMap tools.
* **[OpCore-Simplify](https://github.com/power418/OpCore-Simplify)** — Automated OpenCore EFI deployment and validation framework.

---

## ⚖️ License & Disclaimer

```text
LEGAL DISCLAIMER:
This software and configuration files are distributed on an "AS-IS" basis, without warranties or conditions of any kind.
The authors and contributors accept NO responsibility or liability for hardware damage, permanent data loss,
voided manufacturer warranties, or legal conflicts arising from the use of this repository.
Installing macOS on non-Apple hardware constitutes a breach of Apple's End User License Agreement (EULA).
Deploy this configuration strictly at your own discretion and risk.
```

Distributed under the [MIT License](LICENSE).

<br/>

<div align="center">
  <sub>Engineered with precision for the Hackintosh Community.</sub><br/>
  <b>HP Victus 15-fa0xxx • Intel 12th Gen Alder Lake-H • OpenCore EFI</b>
</div>
