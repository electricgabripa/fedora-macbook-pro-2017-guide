# MacBook Pro 13" 2017 (A1708) Fedora 41 Complete Setup Guide

> **Comprehensive guide to install and configure Fedora 41 on MacBook Pro 2017 without T2 chip**

## 📋 Table of Contents

- [Hardware Specifications](#hardware-specifications)
- [What Works](#what-works)
- [What Doesn't Work](#what-doesnt-work)
- [Installation](#installation)
- [Post-Installation Setup](#post-installation-setup)
  - [Audio Configuration](#audio-configuration)
  - [Wi-Fi Configuration](#wifi-configuration)
  - [Webcam Setup](#webcam-setup)
  - [System Optimizations](#system-optimizations)
  - [Disable Boot Chime](#disable-boot-chime)
  - [Suspend/Resume Issues](#suspendresume-issues)
- [Troubleshooting](#troubleshooting)
- [Credits](#credits)

---

## 🖥️ Hardware Specifications

**Model**: MacBook Pro 13-inch, 2017, Two Thunderbolt 3 ports (MacBookPro14,1)  
**Model Identifier**: A1708  
**Processor**: 2.3 GHz Intel Core i5 dual-core  
**Graphics**: Intel Iris Plus Graphics 640 1536 MB  
**Memory**: 8 GB 2133 MHz LPDDR3  
**Storage**: NVMe SSD

**Important**: This model does **NOT** have the T2 security chip (introduced in 2018+ models).

---

## ✅ What Works

- ✅ **Audio** (internal speakers, headphone jack, internal microphone)
- ✅ **Wi-Fi** (Broadcom BCM4350)
- ✅ **Bluetooth**
- ✅ **Webcam** (FaceTime HD Camera)
- ✅ **Keyboard & Trackpad**
- ✅ **USB-C/Thunderbolt ports**
- ✅ **Battery management**
- ✅ **Brightness controls**
- ✅ **Graphics acceleration**

---

## ❌ What Doesn't Work

- ❌ **Suspend/Resume** (has issues with drivers - see workarounds below)
- ❌ **Hibernation** (problematic with BTRFS swapfile on Fedora 41)

---

## 🚀 Installation

### Prerequisites

1. **Backup your data** if you have macOS installed
2. Download [Fedora 41 Workstation](https://fedoraproject.org/workstation/download)
3. Create bootable USB with [Fedora Media Writer](https://fedoraproject.org/en/workstation/download/) or Balena Etcher

### Installation Steps

1. Insert USB drive and reboot MacBook
2. Hold **Option (⌥)** key during boot
3. Select the USB drive (usually labeled "EFI Boot")
4. Follow Fedora installation wizard
5. **Important**: Choose custom partitioning if you want to keep macOS for firmware updates

**Recommended partition scheme** (if installing Fedora only):
- `/boot/efi` - 500 MB (EFI System Partition)
- `/boot` - 1 GB (ext4)
- `/` - Rest of disk (BTRFS)

---

## ⚙️ Post-Installation Setup

After installing Fedora, update your system first:

```bash
sudo dnf update -y
sudo reboot
```

---

### 🔊 Audio Configuration

The MacBook Pro 2017 uses a **Cirrus Logic CS8409** audio codec that requires a custom driver.

#### Install Dependencies

```bash
sudo dnf install kernel-devel kernel-headers gcc make git
```

#### Install CS8409 Audio Driver

```bash
cd ~
git clone https://github.com/davidjo/snd_hda_macbookpro.git
cd snd_hda_macbookpro
sudo ./install.cirrus.driver.sh
```

#### Reboot

```bash
sudo reboot
```

After reboot, audio should work! Test with:

```bash
speaker-test -c 2
```

#### Note on Kernel Updates

The audio driver may stop working after kernel updates. To fix:

```bash
cd ~/snd_hda_macbookpro
sudo ./install.cirrus.driver.sh
sudo reboot
```

---

### 📡 Wi-Fi Configuration

The MacBook Pro 2017 has a **Broadcom BCM4350** Wi-Fi chip that requires proprietary drivers.

#### Enable RPM Fusion Repositories

```bash
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

#### Install Broadcom Driver

```bash
sudo dnf update
sudo dnf install akmod-wl
```

#### Wait for Module Compilation

```bash
sudo akmods --force
```

#### Reboot

```bash
sudo reboot
```

After reboot, Wi-Fi should work!

#### Verify Wi-Fi is Working

```bash
lsmod | grep wl
ip link show
```

You should see the `wl` module loaded and a wireless interface (e.g., `wlan0`).

---

### 📹 Webcam Setup

The FaceTime HD Camera is connected via PCIe and requires a special driver.

#### Install FaceTime HD Driver

The driver should be automatically installed with recent Fedora kernels. Verify it's working:

```bash
lsmod | grep facetimehd
```

If you see output, the webcam is loaded. Test with:

```bash
sudo dnf install cheese
cheese
```

If the webcam doesn't work, you may need to manually install the `facetimehd` driver from [patjak/facetimehd](https://github.com/patjak/facetimehd).

---

### ⚡ System Optimizations

#### Install TLP for Battery Management

TLP dramatically improves battery life on laptops.

```bash
sudo systemctl mask power-profiles-daemon
sudo dnf install tlp tlp-rdw
sudo systemctl enable tlp.service
sudo systemctl start tlp.service
```

#### Optimize DNF Package Manager

Edit DNF configuration for faster downloads:

```bash
sudo nano /etc/dnf/dnf.conf
```

Add these lines at the end:

```ini
fastestmirror=True
max_parallel_downloads=10
defaultyes=True
keepcache=True
```

Save and exit (CTRL+O, Enter, CTRL+X).

#### Install ZRAM for Better Performance

ZRAM compresses RAM to provide more available memory:

```bash
sudo dnf install zram-generator-defaults
sudo systemctl daemon-reload
sudo systemctl start /dev/zram0
```

#### Enable Firewall

```bash
sudo systemctl enable --now firewalld
```

#### Hardware Video Acceleration in Firefox

1. Open Firefox and go to `about:config`
2. Set `media.ffmpeg.vaapi.enabled` to `true`
3. Set `media.hardware-video-decoding.enabled` to `true`

---

### 🔇 Disable Boot Chime

The MacBook boot chime can be disabled directly from Fedora (even without macOS installed).

#### Disable Boot Chime

```bash
sudo dnf install efivar
sudo chattr -i /sys/firmware/efi/efivars/SystemAudioVolume-7c436110-ab2a-4bbb-a880-fe41995c9f82
printf "\x07\x00\x00\x00\x80" | sudo tee /sys/firmware/efi/efivars/SystemAudioVolume-7c436110-ab2a-4bbb-a880-fe41995c9f82
```

#### Reboot

```bash
sudo reboot
```

The boot chime should now be muted!

#### To Re-enable Boot Chime

Reset NVRAM by holding **Cmd + Option + P + R** during boot for ~20 seconds.

---

### 😴 Suspend/Resume Issues

**Known Issue**: The MacBook Pro 2017 has problems with suspend/resume on Linux. Drivers (especially audio and Wi-Fi) often fail to resume properly, and the screen may stay black.

#### Recommended Solution: Disable Suspend, Use Screen Lock Instead

Instead of suspend, configure the MacBook to just turn off the screen when you close the lid:

```bash
sudo nano /etc/systemd/logind.conf
```

Find and modify these lines (remove the `#` if present):

```ini
HandleLidSwitch=lock
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

Save and reboot:

```bash
sudo reboot
```

**Behavior**:
- When you close the lid: screen turns off and locks, system stays on
- When you open the lid: screen turns on, you enter password
- Wi-Fi, audio, and all drivers continue working perfectly
- Battery drain: ~6-8 hours with screen off (much better than full suspend issues)

#### Alternative: Shutdown on Lid Close

If you prefer complete shutdown:

```ini
HandleLidSwitch=poweroff
```

Fedora boots in 10-15 seconds from an SSD, so this is a viable option.

#### Disable Suspend Completely (Recommended)

To prevent accidental suspend attempts:

```bash
sudo systemctl mask sleep.target suspend.target hybrid-sleep.target
```

---

### 🐛 Troubleshooting

#### Audio Stops Working After Kernel Update

```bash
cd ~/snd_hda_macbookpro
sudo ./install.cirrus.driver.sh
sudo reboot
```

#### Wi-Fi Not Working After Reboot

Check if the module is loaded:

```bash
lsmod | grep wl
```

If not loaded, try:

```bash
sudo modprobe -r wl
sudo modprobe wl
sudo systemctl restart NetworkManager
```

#### "Kernel Tainted" Warning

This is normal when using proprietary drivers (wl, snd_hda_codec_cs8409, facetimehd). The kernel shows a "tainted" warning but everything works fine. This is not a critical error.

#### Audio/Wi-Fi Don't Work After Suspend

This is the known suspend/resume issue. Use the screen lock solution instead of suspend (see [Suspend/Resume Issues](#suspendresume-issues)).

#### Screen Stays Black After Opening Lid

If using suspend and the screen stays black:

1. Press any key or move mouse
2. Try SSH from another device and run: `sudo systemctl restart gdm`
3. If nothing works, force reboot by holding power button

This is why we recommend disabling suspend entirely.

---

## 📚 Additional Resources

- [Fedora Documentation](https://docs.fedoraproject.org/)
- [davidjo/snd_hda_macbookpro](https://github.com/davidjo/snd_hda_macbookpro) - Audio driver
- [RPM Fusion](https://rpmfusion.org/) - Repository for proprietary drivers
- [TLP Documentation](https://linrunner.de/tlp/) - Battery optimization
- [Arch Wiki - MacBookPro14,x](https://wiki.archlinux.org/title/MacBookPro14,x) - General Linux on MacBook info

---

## 🎯 Summary

This guide provides a complete setup for running Fedora 41 on MacBook Pro 13" 2017 (A1708). The most important steps are:

1. ✅ Install CS8409 audio driver for working sound
2. ✅ Install Broadcom Wi-Fi driver via RPM Fusion
3. ✅ Install TLP for battery optimization
4. ✅ Disable suspend and use screen lock instead
5. ✅ Disable boot chime (optional)

With these configurations, your MacBook Pro 2017 will run Fedora smoothly with all essential hardware working!

---

## 👥 Credits

- Audio driver: [davidjo/snd_hda_macbookpro](https://github.com/davidjo/snd_hda_macbookpro)
- Wi-Fi driver: RPM Fusion community
- Testing and documentation: Community contributors
- Special thanks to all Linux on Mac developers and testers

---

## 🙏 Acknowledgments

This guide was created with the invaluable assistance of **Claude (Anthropic AI Assistant)**, who helped troubleshoot every issue, find solutions on GitHub, and document the entire setup process.

Special thanks to Claude for:
- Finding the correct audio driver (snd_hda_macbookpro)
- Debugging Wi-Fi issues with Broadcom BCM4350
- Solving suspend/resume problems
- Creating this comprehensive documentation

Without Claude's help, this guide wouldn't exist! 🤖💙

---

## 📝 License

This guide is released under the MIT License. Feel free to share and modify.

---

## 🤝 Contributing

Found an issue or have improvements? Please open an issue or submit a pull request!

**Tested on**:
- MacBook Pro 13" 2017 (A1708, MacBookPro14,1)
- Fedora 41 Workstation
- Kernel 6.17.10

---

*Last updated: January 2026*
