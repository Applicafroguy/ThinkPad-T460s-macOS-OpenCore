# Thinkpad T460s macOS Catalina (OpenCore bootloader)

> First ever OpenCore build for T460s, everything but SD card reader and Fingerprint scanner works 

<img src="/images/T460s.png" alt="Thinkpad T460s" height="500">

## Introduction

### General knowledge

- To install macOS follow [this guide](https://khronokernel.github.io/Opencore-Vanilla-Desktop-Guide/) by [Mykola Grymalyuk](https://github.com/khronokernel)

- Lots of SSDT patches and how they work can be found [here](https://translate.google.it/translate?sl=zh-CN&tl=en&u=https%3A%2F%2Fgithub.com%2Fdaliansky%2FOC-little)

- Useful tools by [Corpnewt](https://github.com/corpnewt)

- The guys from [Acidanthera](https://github.com/acidanthera) how make this possible

### [Why OpenCore](https://khronokernel.github.io/Opencore-Vanilla-Desktop-Guide/#advantages-of-opencore)

### My Hardware
- Model: Thinkpad T460s (20F9003AUS)
- Processor: Intel Core i7-6600U (2C, 2.6 / 3.4GHz, 4MB)vPro
- Graphics: Integrated Intel HD Graphics 520
- Memory: 4GB Soldered + 4GB DIMM
- Display: 14" WQHD (2560x1440) IPS
- Sound Card: Realtek ALC293
- Multi-touch: None
- Storage: 256GB SSD M.2 Opal2
- Optical: None
- WLAN + Bluetooth: ~~Intel 8260 ac, 2x2 + BT4.1~~ **replaced with [BCM94360CS2](/BCM94360CS2_WLAN_card.md)**
- WWAN: WWAN Upgradable (Legacy_Sierra_QMI.kext needed, not tested but should work)
- Smart Card Reader: None
- Camera: 720p
- Keyboard: Backlit
- Fingerprint Reader: Yes
- Battery: 3-cell (23Wh) + 3-cell (26Wh)

### What if I don't have this exact model?

This EFI will probably work on any T460s regardless of CPU model / RAM amount / Display resolution / Storage drive (SATA or NVMe).

If you happen to have a similiar Thinkpad with Skylake 6th gen Intel processor (like X260, T460, T460p, T560, E560), there is a good chance that this EFI will work on it **with some precaution**:

- double check your DSDT naming (like EC, LPC, KBD, etc.) with provided SSDT naming

- change iGPU inside cofing,plist according to your model

- follow USB ports map and CPU Power Management below


## Recomended changes

### USB ports map

To fix sleep issue I had to built a custom USBPorts.kext, SSDT-UIAC and SSDT-USBX wich maps all available ports on the T460s *except SD card reader and dock links*.
If you need a different map, generate it with [Hackintool](https://github.com/headkaze/Hackintool):

```
use USBInjectAll.kext instead of provided USBPorts.kext (reflect this change in config.plist)
remove SSDT-UIAC and SSDT-USBX
generate those files according to your specific needs with Hackintool
place your files in OC/Kexts and OC/ACPI (reflect this changes in config.plist)
```

### CPU Power Management

I added two kexts to customize CPU profile:

```
CPUFriend.kext
CPUFriendDataProvider.kext
```
[CPUFriendFriend](https://github.com/corpnewt/CPUFriendFriend) was used to set:

```
Low Frequency Mode (LFM) = 800MHz (TDP-down frequency for i7-6600u)
Energy Performance Preference (EPP) = 80 (Balance power)
```

If you have a different CPU model, please, **remove these kexts**, power management is natevely supported by OpenCore. 
In case you want to create your own profile, follow the guide provided for [CPUFriend](https://github.com/acidanthera/CPUFriend)

### Optional

- [Generate your own SMBIOS](https://github.com/corpnewt/GenSMBIOS)
```
run the script with MacbookPro13,1
add results to PlatformInfo -> Generic -> MLB, SystemSerialNumber and SystemUUID
```

- Enable HiDPI with [RDM Utility](https://github.com/usr-sse2/RDM/releases)
```
Install RDM Utility
for 2560x1440 screens I suggest using 1440x810 resolution
to make that simply select 'edit' and use settings as shown below
```
<img src="/images/HiDPI.png" height="300" >

- [Use PrtSc key as Screenshot shortcut](/PrtSc_key_map_to_F13.md)

```
PrtSc is already mapped to F13 in SSDT-PS2K
```

- Monitor temperatures and power consumption with [HWMonitor](https://github.com/kzlekk/HWSensors/releases)
```
I know it's old and no longer supported, but it gets the job done and i really like the simple look
```

- Make dock animation faster and without delay
```
defaults write com.apple.dock autohide-delay -float 0
defaults write com.apple.dock autohide-time-modifier -float 0.5
killall Dock
```

## Bios settings

- `Security -> Security Chip` **Disabled**
- `Memory Protection -> Execution Prevention` **Enabled**
- `Virtualization -> Intel Virtualization Technology` **Enabled**
- `Virtualization -> Intel VT-d Feature` **Disabled**
- `Anti-Theft -> Current Setting` **Disabled**
- `Anti-Theft -> Computrace -> Current Setting` **Disabled**
- `Secure Boot -> Secure Boot` **Disabled**
- `Intel SGX -> Intel SGX Control` **Disabled**
- `Device Guard` **Disabled**
- `UEFI/Legacy Boot` **UEFI Only**
- `CSM Support` **No**

## What's working

>[Boot time from OC Picker to Desktop is 26s](https://www.youtube.com/watch?v=SnuQjuIrfc0)

- CPU Power Management
    - `<1W on IDLE`

- All USB ports
    - `custom USBPorts kext is used, make a new one if using dock`

- Internal camera

- Sleep / Wake / Shutdown / Reboot
    - `custom USBPorts.kext required`

- Ethernet

- **[Wifi, Bluetooth, Airdrop, Handoff, Continuity, Sidecar wireless](/BCM94360CS2_WLAN_card.md)**
    - `A guide is provided`

- iMessage, FaceTime, App Store, iTunes Store 
    - `Generate your on SMBIOS`

- Audio in/out
    - `audio trough dock should work too thanks to AppleALC tluck's layout 28`

- Battery **(very stable and precise capacity tracking)**
    - `Thanks to EchoEsprit work for T450s`

- Keyboard
    - `volume and brighness hotkeys`

- Trackpad, Trackpoint and physical buttons
    - `two fingers swipe and tree fingers gestures`

- miniDP and HDMI
    - `video signal trough dock should work too thank to links added in DevicesProperty`

- Internal camera
    - `works without additional files`

- SIP and FileVault 2 can be enabled
    - `Are disabled in config.plist`

## What's not working

> If you have any questions or suggestions feel free to contact me

- SD Card Reader
    - `I will try to make it works sometime in the future`

- Fingerprint Reader
    - `Don't think it will ever be working on macOS`

- Wake on WiFi cause the transfer rate to be reduced after sleep
    - to fix that `SystemPreferences > Energy Saver > Power Adapter > Wake for Wi-fi network access`  **Disabled**

## Update tracker

| Item | Version |
| :--- | :--- |
| **MacOS** | 10.15.4 |
| **OpenCore** | 0.5.7 |
| **Lilu** | 1.4.3 |
| **VirtualSMC** | 1.1.1 |
| **WhateverGreen** | 1.3.8 |
| **AppleALC** | 1.4.8 |
| **VoodooPS2Controller** | 2.1.3 |
| **IntelMausi** | 1.0.2 |


## If you found my work useful please consider a PayPal donation

<a href="https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=Y5BE5HYACDERG&source=url" target="_blank"><img src="/images/buymeacoffee.png" alt="Buy Me A Coffee" width="300" ></a>

## Thanks to

All the hackintosh community, especially you guys on GitHub.
