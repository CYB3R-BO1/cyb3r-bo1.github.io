---
title: Setting Up Kali Linux on VirtualBox
date: 2026-01-31 15:30:00 +0530
categories: [Walkthrough]
tags: [cybersecurity, walkthrough]     # TAG names should always be lowercase
description: This post consists walkthrough to setup Kali Linux on VirtualBox"
---

This guide walks through **two ways** to set up Kali Linux on VirtualBox:

1. **Using the Pre-built Virtual Machine (Recommended)**
2. **Using the Kali Linux ISO (Manual Installation)**

I’ll also cover **Guest Additions installation** for better performance and full-screen support.

---

## TL;DR

- If you want **fastest setup** → use **Pre-built Kali VM**
- If you want to **learn Linux installation internals** → use **ISO**
- Always install **Guest Additions** for:
    - Full-screen support
    - Better resolution
    - Smoother performance

**Jump to section:**

- Pre-built VM Setup
- Install Guest Additions
- ISO Installation

---

## Requirements

- A system with **Virtualization enabled** (Intel VT-x / AMD-V)
- At least **8 GB RAM** recommended (4 GB minimum)
- **VirtualBox** installed on host OS

---

## Method 1: Setting Up Kali Linux Using Pre-built Virtual Machine

This is the **recommended approach** for most users.

### Step 1: Download Kali Linux (Pre-built VM)

Download Kali Linux for VirtualBox from the official site:

- https://www.kali.org/get-kali/#kali-virtual-machines

Direct download (example version):

- https://cdimage.kali.org/kali-2025.4/kali-linux-2025.4-virtualbox-amd64.7z

**Why Pre-built VM?**

- Minimal setup
- No manual partitioning
- Ready-to-use environment

---

### Step 2: Install VirtualBox

Download and install VirtualBox based on your host OS:

- https://www.virtualbox.org/wiki/Downloads

Install **VirtualBox Platform Package** (Extension Pack is optional but recommended).

---

### Step 3: Extract Kali VM Files

Extract the downloaded `.7z` file.

You’ll see two important files:

- **`.vdi`** → Virtual Disk Image (the actual OS disk)
- **`.vbox`** → VirtualBox Machine Definition (VM configuration)

---

### Step 4: Import Kali into VirtualBox

1. Open **VirtualBox**
2. Click **Machine → Add**
3. Select the `.vbox` file
4. Kali Linux will appear in your VM list

---

### Step 5: Recommended VM Settings (Before Boot)

Select the Kali VM → **Settings**

#### General → Advanced

- Disable **Drag’n’Drop**
- Disable **Clipboard Sharing** (optional, security hygiene)

#### System → Processor

- Allocate **as many CPUs as your system allows** (without starving host OS)

#### Display → Screen

- Set **Video Memory to maximum**

#### Storage

- Ensure disk type is **Dynamically allocated**
    - This prevents wasting unused disk space

#### Network (Optional)

- If you face network issues later:
    - Change **Adapter Type**
    - Switch between **NAT** and **Bridged**

---

### Step 6: Start Kali Linux

Start the VM.

**Default credentials:**

```
username: kali
password: kali
```

---

## Installing VirtualBox Guest Additions

Without Guest Additions, you’ll face:

- Low resolution
- No full-screen mode
- Laggy UI

### Step 1: Update Kali

```bash
sudo apt update
sudo apt upgrade -y
sudo reboot
```

---

### Step 2: Install Linux Headers

```bash
sudo apt install -y linux-headers-$(uname -r)
```

---

### Step 3: Insert Guest Additions CD

From the VirtualBox menu:

```
Devices →InsertGuestAdditionsCDImage
```

---

### Step 4: Copy Guest Additions Files

1. Open **File Manager (Thunar)**
2. Go to **Devices → VBox_GAs**
3. Copy all files
4. Paste them into a directory (e.g., `~/Downloads/VBoxGA`)

---

### Step 5: Install Guest Additions

```bash
cd ~/Downloads/VBoxGA
ls *.run
sudochmod +x VBoxLinuxAdditions.run
sudo ./VBoxLinuxAdditions.run
sudo reboot
```

After reboot, enable **Full Screen Mode**.

Kali Linux setup complete.

---

## Method 2: Setting Up Kali Linux Using Kali Linux ISO

This method is **slower**, but educational.

### Step 1: Download VirtualBox

- https://www.virtualbox.org/wiki/Downloads

---

### Step 2: Download Kali Linux Installer ISO

- https://www.kali.org/get-kali/#kali-installer-images

Download **Installer ISO** (not Live).

---

### Step 3: Create a New Virtual Machine

1. Open VirtualBox → **New**
2. Name: `Kali Linux`
3. Type: `Linux`
4. Version: `Debian (64-bit)`
5. Select downloaded `.iso`

---

### Step 4: Allocate Resources

- RAM: **4-8 GB**
- CPUs: **2-4**
- Disk: **Dynamically allocated**, ≥ **40 GB**

---

### Step 5: Start Installation

1. Click **Start**
2. Select **Graphical Install**
3. Choose:
    - Language
    - Country
    - Keyboard
4. Set:
    - Hostname
    - Username
    - Password

---

### Step 6: Disk & Software Selection

- Accept default partitioning
- Choose **Yes** to write changes
- Leave **software selection default**
- Continue installation

---

### Step 7: Finish Installation

Once complete:

- Reboot
- Remove ISO
- Log in with your credentials

**Install Guest Additions using the same steps as above**

---

## Common Issues & Fixes

- **Black screen / low resolution**
    - Install Guest Additions
- **No internet**
    - Switch network adapter to NAT
- **Debian (64-bit) not showing**
    - Enable virtualization in BIOS

---

## Final Notes

- Use **Pre-built VM** unless you *want* to learn Linux installation
- Always update Kali after first boot

Your Kali Linux is now ready to use.

_Plus Ultra._