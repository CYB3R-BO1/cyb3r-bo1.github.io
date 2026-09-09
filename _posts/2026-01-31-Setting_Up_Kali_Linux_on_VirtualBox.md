---
title: Setting Up Kali Linux on VirtualBox
date: 2026-01-31 16:40:00 +0530
categories: [Walkthrough]
tags: [virtualization, kali-linux, virtualbox]
description: How I set up Kali Linux on VirtualBox — both the quick way (pre-built VM) and the manual way (ISO install), plus Guest Additions, a few hardening steps I actually use, and the issues that always seem to come up.
---

I've set up Kali on VirtualBox more times than I can count. Sometimes for a quick CTF, sometimes to test something sketchy in isolation, sometimes just to have a Linux environment handy. This post covers the two ways I do it, the settings that actually matter, and the gotchas that waste time.

---

## The Short Version

**Use the pre-built VM** if you want to be up and running in 10 minutes. It's what I do 90% of the time.

**Use the ISO** if you want to learn the installer, need encrypted LVM, or want a minimal base. Takes longer but you'll understand your system better.

Either way, **install Guest Additions**. Full-screen, clipboard sharing, drag-and-drop — it's not optional if you want a usable desktop.

---

## What You'll Need

| Spec | Minimum | What I Actually Use |
|------|---------|---------------------|
| RAM | 4 GB | 16 GB (8 GB works fine) |
| CPU | 2 cores | 4-6 cores, VT-x/AMD-V enabled |
| Disk | 25 GB | 60-80 GB dynamic |
| Video RAM | 128 MB | 256 MB |
| VirtualBox | 7.0+ | Latest stable |

**Virtualization must be on in BIOS.** If you only see 32-bit OS options in VirtualBox, that's why.

---

## Method 1: Pre-built VM (The Way I Usually Do It)

### 1. Grab the Image

Official downloads: [kali.org/get-kali/#kali-virtual-machines](https://www.kali.org/get-kali/#kali-virtual-machines)

Direct link changes with each release, but looks like:
```
https://cdimage.kali.org/kali-2026.1/kali-linux-2026.1-virtualbox-amd64.7z
```

Verify the SHA256. I've had corrupted downloads before — saves headache later.

```bash
sha256sum kali-linux-2026.1-virtualbox-amd64.7z
# Compare with the checksum on the downloads page
```

### 2. VirtualBox + Extension Pack

Download VirtualBox from [virtualbox.org](https://www.virtualbox.org/wiki/Downloads). Install the platform package for your OS.

**Extension Pack** — grab it from the same page. You want USB 3.0 support, disk encryption, PXE boot. In VirtualBox: Tools → Extension Pack Manager → Install.

### 3. Extract and Import

```bash
7z x kali-linux-2026.1-virtualbox-amd64.7z
# Windows: 7-Zip or PeaZip
```

You'll get a `.vdi` (the disk) and a `.vbox` (the config). Double-click the `.vbox` file — it imports automatically. Or Machine → Add in VirtualBox.

### 4. Settings I Change Before First Boot

Right-click the VM → Settings:

**System → Motherboard**
- Base Memory: 8192 MB (or whatever you can spare)
- Boot Order: Optical first, then Hard Disk. Disable Floppy/Network.
- EFI: **Off** — pre-built uses BIOS/MBR.

**System → Processor**
- 4-6 CPUs (leave 2 for host)
- PAE/NX: enabled
- Execution Cap: 100%

**Display → Screen**
- Video Memory: 256 MB (max)
- Graphics Controller: **VMSVGA** — this matters. VBoxVGA and VBoxSVGA both cause issues on Kali.
- 3D Acceleration: On

**Storage**
- SATA controller, check "Solid State Drive" if your host has an SSD
- "Use Host I/O Cache" — helps with I/O latency

**Network**
- **NAT** for general use (updates, browsing)
- **Bridged** when I need the VM on the LAN (Nmap, Responder, Bettercap, etc.)
- I switch between them depending on what I'm doing

**USB**
- USB 3.0 (xHCI) Controller enabled
- Add filters for any WiFi adapters / hardware tokens you'll pass through

### 5. First Boot

Default login:
```
kali / kali
```

**Change the password immediately.**
```bash
passwd
```

Then update:
```bash
sudo apt update && sudo apt full-upgrade -y && sudo reboot
```

While that's running, install a few basics:
```bash
sudo apt install -y git curl wget vim tmux htop net-tools \
    build-essential python3-pip golang-go
```

Set your timezone:
```bash
sudo timedatectl set-timezone Asia/Kolkata  # or whatever
```

---

## Method 2: ISO Install (When I Want Control)

### 1. Download the Installer ISO

Not the Live ISO — the **Installer** one:
[kali.org/get-kali/#kali-installer-images](https://www.kali.org/get-kali/#kali-installer-images)

Verify SHA256.

### 2. Create the VM

VirtualBox → New (Ctrl+N)
- Name: Kali Linux
- ISO: select the downloaded file
- **Skip Unattended Installation** — I want control
- Username/password/hostname: your call
- Resources: same as above (8-16 GB RAM, 4+ CPUs, 60+ GB dynamic disk)

### 3. The Installer Walkthrough

Graphical Install → follow prompts. A few decisions:

**Partitioning:** Guided - use entire disk is fine for most. If you want encrypted LVM, choose Manual. I've done both — encrypted LVM is worth it if the VM might leave your machine.

**Software Selection:** This is where people miss things.
- `kali-linux-default` (pre-selected)
- `kali-linux-large` — **I always pick this**. More tools, less "apt install" later.
- `kali-linux-everything` — only if you have 100+ GB and patience
- **`virtualbox-guest-x11`** — check this. Guest Additions installs during OS setup. One less step later.

**GRUB:** Install to `/dev/sda` (the virtual disk).

### 4. Post-Install

Reboot, eject the ISO (Devices → Optical Drives → Remove), log in, update:
```bash
sudo apt update && sudo apt full-upgrade -y && sudo reboot
```

---

## Guest Additions — Do This Either Way

Without it: tiny window, no clipboard sharing, no drag-drop, annoying.

### The Easy Way (Package Manager)

```bash
sudo apt update && sudo apt full-upgrade -y
sudo reboot

sudo apt install -y linux-headers-$(uname -r) build-essential dkms
sudo apt install -y virtualbox-guest-dkms virtualbox-guest-x11 virtualbox-guest-utils

sudo /sbin/vboxconfig
sudo reboot
```

This is the **better way**. DKMS rebuilds kernel modules automatically when the kernel updates. The CD image method breaks every kernel upgrade.

### Verify It Worked

```bash
lsmod | grep -E 'vboxguest|vboxsf|vboxvideo'
systemctl status vboxadd-service
```

Full-screen: View → Full-screen Mode (Host+F). Clipboard: Devices → Shared Clipboard → Bidirectional.

### Shared Folders (If You Need Them)

VM Settings → Shared Folders → Add your host folder. Auto-mount, Make Permanent.

In the VM:
```bash
sudo usermod -aG vboxsf $USER
# logout/login or reboot
ls /media/sf_YourFolderName
```

---

## Hardening — The Stuff I Actually Do

Not a full CIS benchmark. Just the basics before I connect to anything untrusted.

### 1. Passwords
```bash
passwd                    # your user
sudo passwd root          # if you set one during ISO install
```

### 2. SSH Keys Only
```bash
# On host:
ssh-keygen -t ed25519 -C "kali-vm"
ssh-copy-id kali@<vm-ip>

# On VM:
sudo vim /etc/ssh/sshd_config
# PermitRootLogin no
# PasswordAuthentication no
# PubkeyAuthentication yes
# MaxAuthTries 3
# ClientAliveInterval 300

sudo systemctl restart ssh
sudo systemctl enable ssh
```

### 3. UFW — Simple Firewall
```bash
sudo apt install -y ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow from 192.168.0.0/16 to any port 22  # adjust your subnet
sudo ufw enable
```

### 4. Kill Services I Don't Need
```bash
systemctl list-unit-files --state=enabled | grep -E 'bluetooth|cups|avahi'
sudo systemctl disable --now bluetooth.service cups.service avahi-daemon.service
```

### 5. Unattended Security Updates
```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades  # say Yes
```

### 6. Kernel Params (sysctl)
Create `/etc/sysctl.d/99-kali-hardening.conf`:
```ini
# Network
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.ip_forward = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
net.ipv4.tcp_syncookies = 1
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

# Kernel
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 2
kernel.yama.ptrace_scope = 1
fs.suid_dumpable = 0
```
```bash
sudo sysctl --system
```

### 7. Quick Audit
```bash
sudo apt install -y lynis
sudo lynis audit system
```
Fix whatever it flags that matters to you.

---

## Performance Tweaks

### Inside the VM
```bash
# SSD TRIM for the virtual disk
sudo systemctl enable fstrim.timer
sudo systemctl start fstrim.timer

# Less swap thrashing
echo 'vm.swappiness=10' | sudo tee /etc/sysctl.d/99-swappiness.conf
sudo sysctl --system

# ZRAM — compressed RAM swap, way faster than disk
sudo apt install -y zram-tools
sudo systemctl enable --now zramswap

# Preload learns what you run and caches it
sudo apt install -y preload

# Cleanup
sudo apt autoremove -y && sudo apt autoclean
sudo journalctl --vacuum-time=7d
```

### VirtualBox Side
- Host I/O Cache: **On** (Storage → Controller)
- Nested Paging: **On** (System → Acceleration)
- Paravirtualization: KVM (Linux host) / Hyper-V (Windows host)
- Large Pages: On if host has 16+ GB RAM

---

## Problems I've Actually Hit

| Symptom | What Fixed It |
|---------|---------------|
| Only 32-bit OS types | Enable VT-x/AMD-V in BIOS |
| Black screen on boot | Display → Graphics Controller → **VMSVGA** |
| Stuck at 800x600 | Guest Additions not installed / `vboxvideo` module missing |
| No internet on NAT | `sudo dhclient -v eth0` or just reboot |
| No internet on Bridged | Wrong physical adapter selected in Network settings |
| Clipboard broken | `sudo systemctl restart vboxadd-service` |
| Kernel headers mismatch after update | `sudo apt install linux-headers-$(uname -r) && sudo /sbin/vboxconfig` |
| VM feels sluggish | More RAM/CPU, VMSVGA, 3D accel on, Host I/O Cache on |
| Clock drift | `sudo apt install -y chrony && sudo systemctl enable --now chronyd` |
| Shared folder permission denied | `sudo usermod -aG vboxsf $USER` → reboot |
| "No bootable medium" on ISO boot | Storage → attach ISO, move Optical to top of boot order |

---

## Snapshots — Your Safety Net

After Guest Additions + hardening + updates, **take a snapshot**. VM powered off → Snapshots tab → Take (Ctrl+Shift+T). Name it something like "Baseline - hardened + GA + updated".

Before anything risky (CTF, malware sample, exploit dev):
```bash
# On host
VBoxManage snapshot "Kali Linux" take "Pre-CTF-$(date +%F)"
# To restore later:
VBoxManage snapshot "Kali Linux" restore "Pre-CTF-2026-01-31"
```

Export to OVA for portable backups:
```bash
VBoxManage export "Kali Linux" --output kali-baseline.ova --ovf20
```

---

## Aliases I Keep in `~/.bashrc`

```bash
# System
alias update='sudo apt update && sudo apt full-upgrade -y && sudo apt autoremove -y && sudo apt autoclean'
alias ports='ss -tulnp'
alias myip='curl -s ifconfig.me'

# VM conveniences
alias vm-shared='ls /media/sf_*'
alias vm-clipboard='vboxclient --clipboard'

# Quick checks
alias ssh-status='systemctl status ssh'
alias ufw-status='sudo ufw status verbose'
```

---

## Final Thoughts

- **Pre-built VM** gets you working fastest. Use it.
- **ISO** teaches you the installer and lets you encrypt. Worth doing once.
- **Guest Additions via apt** (dkms packages) saves you from re-installing after every kernel update.
- **Snapshot before breaking things.** I learned this the hard way.
- **Harden before you connect** to anything you don't trust.

---

## Links I Keep Bookmarked

- [Kali Docs](https://www.kali.org/docs/)
- [VirtualBox Manual](https://www.virtualbox.org/manual/UserManual.html)
- [Kali Tools](https://www.kali.org/tools/)
- [Lynis](https://cisofy.com/lynis/)

---

_Plus Ultra._