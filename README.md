<p align="center">
  <img src="https://netpilot.ir/assest/img/luks-cheet-sheet.png" alt="LUKS Cheat Sheet Banner" width="500" height: 100%;>
</p>

<h1 align="center">LUKS Cheat Sheet</h1>

<p align="center">Complete Guide to Linux Disk Encryption with cryptsetup</p>

# LUKS Cheat Sheet – Complete cryptsetup Guide for Linux Disk Encryption

✅ Full LUKS1 & LUKS2 Guide  
✅ Complete cryptsetup Commands  
✅ Password & Keyslot Management  
✅ Keyfile Authentication  
✅ LUKS Header Backup & Restore  
✅ Auto Unlock with crypttab & fstab  
✅ File-Based Encrypted Containers  
✅ LPIC-2 & LPIC-3 Ready  

> This is a complete and professional **LUKS Cheat Sheet** written for Linux system administrators, DevOps engineers, and security students.

---

## 📑 Table of Contents

- [What is LUKS?](#-what-is-luks)
- [Basics & Concepts](#1-basics--concepts)
- [Installation](#2-installation)
- [Formatting a Disk with LUKS](#3-formatting-a-disk-with-luks-all-data-erased)
- [Open & Close](#4-open--close-unlock--lock)
- [Create Filesystem & Mount](#5-create-filesystem--mount)
- [Password & Keyslot Management](#6-password--keyslot-management)
- [Keyfile Authentication](#7-keyfile-authentication)
- [LUKS Header Backup & Restore](#8-luks-header-backup--restore-critical)
- [Resize Encrypted Device](#9-resize-encrypted-device)
- [Status, UUID & Recovery](#10-status-uuid--recovery)
- [Auto Unlock at Boot](#11-auto-unlock-at-boot-crypttab--fstab)
- [File-Based LUKS Container](#12-file-based-luks-container)
- [Suspend & Resume](#13-suspend--resume-ram-lock)
- [Security Best Practices](#14-security-best-practices)
- [Author](#-author)

---

## 🔍 SEO Keywords

LUKS cheat sheet  
cryptsetup cheat sheet  
linux disk encryption  
luks encryption guide  
luks header backup  
luks keyfile  
luks fstab crypttab  
luks full disk encryption  
luks tutorial  

---

## 📌 What is LUKS?

**LUKS (Linux Unified Key Setup)** is the standard disk encryption system for Linux.  
It provides strong encryption at the block-device level and is widely used for:

- Full Disk Encryption (FDE)
- Encrypted partitions
- Encrypted USB drives
- Secure containers
- Encrypted virtual machines

---

# ✅ LUKS / cryptsetup – Ultimate Complete Cheat Sheet

---

## 1. Basics & Concepts

- LUKS → Linux disk encryption standard  
- cryptsetup → LUKS management tool  
- Block device encryption → `/dev/sdX`, `/dev/nvmeX`, LVM, RAID  
- Keyslot → Each password stored in a separate slot  
- LUKS1 → Legacy compatibility  
- LUKS2 → Modern, secure, flexible metadata  
- Mapping name → `/dev/mapper/<name>`

---

## 2. Installation

### Debian / Ubuntu / Kali
```bash
sudo apt update
sudo apt install cryptsetup
````

### RHEL / CentOS / Rocky / Alma

```bash
sudo dnf install cryptsetup
```

### Arch Linux

```bash
sudo pacman -S cryptsetup
```

Check version:

```bash
cryptsetup --version
```

---

## 3. Formatting a Disk with LUKS (ALL DATA ERASED)

### Default (LUKS2)

```bash
sudo cryptsetup luksFormat /dev/sdX1
```

### Force Version

```bash
sudo cryptsetup luksFormat --type luks1 /dev/sdX1
sudo cryptsetup luksFormat --type luks2 /dev/sdX1
```

### Custom Encryption

```bash
sudo cryptsetup luksFormat \
  --cipher aes-xts-plain64 \
  --key-size 512 \
  --hash sha256 \
  --iter-time 5000 \
  /dev/sdX1
```

⚠️ Non-interactive (NOT recommended):

```bash
echo "password" | sudo cryptsetup luksFormat /dev/sdX1 -
```

---

## 4. Open & Close (Unlock / Lock)

### Open

```bash
sudo cryptsetup open /dev/sdX1 secure
```

Result:

```
/dev/mapper/secure
```

### Close

```bash
sudo cryptsetup close secure
```

### Read Only

```bash
sudo cryptsetup open --readonly /dev/sdX1 secure
```

---

## 5. Create Filesystem & Mount

```bash
sudo mkfs.ext4 /dev/mapper/secure
sudo mkdir -p /mnt/secure
sudo mount /dev/mapper/secure /mnt/secure
```

Unmount:

```bash
sudo umount /mnt/secure
```

---

## 6. Password & Keyslot Management

View info:

```bash
sudo cryptsetup luksDump /dev/sdX1
```

Add password:

```bash
sudo cryptsetup luksAddKey /dev/sdX1
```

Remove password:

```bash
sudo cryptsetup luksRemoveKey /dev/sdX1
```

Remove specific slot:

```bash
sudo cryptsetup luksKillSlot /dev/sdX1 1
```

Change password:

```bash
sudo cryptsetup luksChangeKey /dev/sdX1
```

---

## 7. Keyfile Authentication

Create keyfile:

```bash
sudo dd if=/dev/urandom of=/root/luks.key bs=64 count=1
sudo chmod 600 /root/luks.key
```

Add keyfile:

```bash
sudo cryptsetup luksAddKey /dev/sdX1 /root/luks.key
```

Unlock with keyfile:

```bash
sudo cryptsetup open /dev/sdX1 secure --key-file /root/luks.key
```

---

## 8. LUKS Header Backup & Restore (CRITICAL)

Backup:

```bash
sudo cryptsetup luksHeaderBackup /dev/sdX1 \
--header-backup-file /root/luks-header.img
```

Restore:

```bash
sudo cryptsetup luksHeaderRestore /dev/sdX1 \
--header-backup-file /root/luks-header.img
```

⚠️ Wrong restore = permanent data loss!

---

## 9. Resize Encrypted Device

Resize mapping:

```bash
sudo cryptsetup resize secure
```

Resize filesystem:

```bash
sudo e2fsck -f /dev/mapper/secure
sudo resize2fs /dev/mapper/secure
```

---

## 10. Status, UUID & Recovery

Status:

```bash
sudo cryptsetup status secure
```

LUKS UUID:

```bash
sudo cryptsetup luksUUID /dev/sdX1
```

Filesystem UUID:

```bash
sudo blkid /dev/mapper/secure
```

Repair:

```bash
sudo e2fsck -f /dev/mapper/secure
```

---

## 11. Auto Unlock at Boot (crypttab & fstab)

### /etc/crypttab

```text
secure UUID=<LUKS_UUID> none luks
```

With keyfile:

```text
secure UUID=<LUKS_UUID> /root/luks.key luks
```

### /etc/fstab

```text
/dev/mapper/secure /mnt/secure ext4 defaults 0 2
```

Or with UUID:

```text
UUID=<FS_UUID> /mnt/secure ext4 defaults 0 2
```

---

## 12. File-Based LUKS Container

Create file:

```bash
dd if=/dev/urandom of=secure.img bs=1M count=2048
```

Encrypt:

```bash
sudo cryptsetup luksFormat secure.img
```

Open:

```bash
sudo cryptsetup open secure.img securefile
```

Create filesystem:

```bash
sudo mkfs.ext4 /dev/mapper/securefile
sudo mount /dev/mapper/securefile /mnt/securefile
```

Close:

```bash
sudo umount /mnt/securefile
sudo cryptsetup close securefile
```

---

## 13. Suspend & Resume (RAM Lock)

Suspend:

```bash
sudo cryptsetup luksSuspend secure
```

Resume:

```bash
sudo cryptsetup luksResume secure
```

---

## 14. Security Best Practices

✅ Always backup LUKS header
✅ Use strong passwords (16+ characters)
✅ Never store keyfile on same disk
✅ Encrypt swap partition
✅ Avoid passwords in command history
✅ Always test on VM first
✅ Prefer LUKS2
✅ Use AES-XTS-512

---

## 👨‍💻 Author

Created by **Mahdi Norouzi**
Linux Administrator & DevOps Candidate

🌐 Website: [https://netpilot.ir](https://netpilot.ir)
📂 GitHub: [https://github.com/](https://github.com/)

---

⭐ If this repository helps you, please give it a **star** to support the project!
