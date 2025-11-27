# luks-cheatsheet
Complete LUKS Cheat Sheet for Linux Disk Encryption
# LUKS Cheat Sheet – Complete cryptsetup Guide for Linux Disk Encryption

✅ Full LUKS2 & LUKS1 Guide  
✅ cryptsetup Commands Explained  
✅ Password, Keyfile, Header Backup  
✅ Auto Mount with crypttab & fstab  
✅ File Based Encryption  
✅ LPIC-2 & LPIC-3 Ready  

---

## 🔐 What is LUKS?
LUKS (Linux Unified Key Setup) is the standard disk encryption system for Linux.

This repository provides a **complete, professional, and practical LUKS Cheat Sheet**.

---

## 📂 Download Cheat Sheet
➡️ [Download LUKS Cheat Sheet](./luks-cheatsheet.md)

---

## 🔍 SEO Keywords
LUKS cheat sheet  
cryptsetup cheat sheet  
linux disk encryption  
luks encryption guide  
luks header backup  
luks keyfile  
luks fstab crypttab  

---

## 🌍 Author
Created by **Mahdi Norouzi**  
Linux & DevOps Engineer Candidate  

---

⭐ If this helped you, give the repo a star!




```md
# LUKS / cryptsetup – Ultimate Complete Cheat Sheet

> Linux Unified Key Setup – Full-disk and block-device encryption standard on Linux

---

## Table of Contents

1. Basics & Concepts  
2. Installation  
3. Formatting a Disk with LUKS  
4. Opening (Unlock) & Closing (Lock)  
5. Creating a Filesystem  
6. Password & Keyslot Management  
7. Keyfile Management  
8. LUKS Header Backup & Restore  
9. Resizing Encrypted Devices  
10. Status, UUID & Troubleshooting  
11. crypttab & fstab (Auto Mount at Boot)  
12. File-Based LUKS (Encrypted Container File)  
13. Suspend & Resume (Memory Lock)  
14. Security Best Practices  

---

## 1. Basics & Concepts

- **LUKS** → Linux Unified Key Setup (standard disk encryption layer)
- **cryptsetup** → Userspace tool to manage LUKS
- **Block Device Encryption** → Works on `/dev/sdX`, `/dev/nvmeX`, LVM, RAID
- **Keyslot** → Each password/key stored in a separate slot (0–7 or more)
- **LUKS1** → Legacy, maximum compatibility
- **LUKS2** → Modern, more secure, flexible metadata
- **Mapping Name** → Logical unlocked device under:
  
```

/dev/mapper/<name>

````

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

Verify installation:

```bash
cryptsetup --version
```

---

## 3. Formatting a Disk with LUKS (DATA WILL BE ERASED)

### Basic Format (Default = LUKS2 on modern systems)

```bash
sudo cryptsetup luksFormat /dev/sdX1
```

### Force LUKS Version

```bash
# LUKS2
sudo cryptsetup luksFormat --type luks2 /dev/sdX1

# LUKS1
sudo cryptsetup luksFormat --type luks1 /dev/sdX1
```

### Custom Cipher & Security Settings

```bash
sudo cryptsetup luksFormat \
  --cipher aes-xts-plain64 \
  --key-size 512 \
  --hash sha256 \
  --iter-time 5000 \
  /dev/sdX1
```

* `aes-xts-plain64` → Recommended disk cipher
* `key-size 512` → Stronger encryption
* `iter-time` → PBKDF computation delay (ms)

### Non-Interactive Mode (For Scripts – NOT RECOMMENDED)

```bash
echo "MyPassword" | sudo cryptsetup luksFormat /dev/sdX1 -
```

⚠️ Password may leak via history & process list.

---

## 4. Opening (Unlock) & Closing (Lock)

### Unlock Encrypted Device

```bash
sudo cryptsetup open /dev/sdX1 secure
```

Result:

```
/dev/mapper/secure
```

### Close Device

```bash
sudo cryptsetup close secure
```

### Read-Only Mode

```bash
sudo cryptsetup open --readonly /dev/sdX1 secure
```

---

## 5. Creating a Filesystem

After unlocking:

```bash
sudo mkfs.ext4 /dev/mapper/secure
```

Mount it:

```bash
sudo mkdir -p /mnt/secure
sudo mount /dev/mapper/secure /mnt/secure
```

Unmount:

```bash
sudo umount /mnt/secure
```

---

## 6. Password & Keyslot Management

### View LUKS Metadata & Keyslots

```bash
sudo cryptsetup luksDump /dev/sdX1
```

### Add New Password

```bash
sudo cryptsetup luksAddKey /dev/sdX1
```

### Remove Password (By Entering It)

```bash
sudo cryptsetup luksRemoveKey /dev/sdX1
```

### Remove Specific Slot

```bash
sudo cryptsetup luksKillSlot /dev/sdX1 1
```

### Change Password

```bash
sudo cryptsetup luksChangeKey /dev/sdX1
```

---

## 7. Keyfile Management

### Create a Secure Keyfile

```bash
sudo dd if=/dev/urandom of=/root/luks.key bs=64 count=1
sudo chmod 600 /root/luks.key
```

### Add Keyfile to LUKS

```bash
sudo cryptsetup luksAddKey /dev/sdX1 /root/luks.key
```

### Unlock Using Keyfile

```bash
sudo cryptsetup open /dev/sdX1 secure --key-file /root/luks.key
```

---

## 8. LUKS Header Backup & Restore (CRITICAL)

### Backup Header

```bash
sudo cryptsetup luksHeaderBackup /dev/sdX1 \
--header-backup-file /root/luks-header.img
```

### Restore Header (DANGEROUS!)

```bash
sudo cryptsetup luksHeaderRestore /dev/sdX1 \
--header-backup-file /root/luks-header.img
```

⚠️ Wrong restore = permanent data loss.

---

## 9. Resizing Encrypted Device

### Resize LUKS Mapping

```bash
sudo cryptsetup resize secure
```

### Resize Filesystem (ext4)

```bash
sudo e2fsck -f /dev/mapper/secure
sudo resize2fs /dev/mapper/secure
```

---

## 10. Status, UUID & Troubleshooting

### Device Status

```bash
sudo cryptsetup status secure
```

### LUKS UUID

```bash
sudo cryptsetup luksUUID /dev/sdX1
```

### Filesystem UUID

```bash
sudo blkid /dev/mapper/secure
```

### Filesystem Repair

```bash
sudo e2fsck -f /dev/mapper/secure
```

---

## 11. crypttab & fstab (Auto Unlock at Boot)

### /etc/crypttab

Without keyfile:

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

Or using UUID:

```text
UUID=<FS_UUID> /mnt/secure ext4 defaults 0 2
```

---

## 12. File-Based LUKS (Encrypted Container File)

### Create File

```bash
dd if=/dev/urandom of=secure.img bs=1M count=2048
```

### Encrypt File

```bash
sudo cryptsetup luksFormat secure.img
```

### Open File

```bash
sudo cryptsetup open secure.img securefile
```

### Create Filesystem

```bash
sudo mkfs.ext4 /dev/mapper/securefile
sudo mount /dev/mapper/securefile /mnt/securefile
```

### Close

```bash
sudo umount /mnt/securefile
sudo cryptsetup close securefile
```

---

## 13. Suspend & Resume (Memory Locking)

### Suspend Device (RAM Lock)

```bash
sudo cryptsetup luksSuspend secure
```

### Resume Device

```bash
sudo cryptsetup luksResume secure
```

---

## 14. Security Best Practices

* ✅ Always backup LUKS header
* ✅ Use long passphrases (16+ characters)
* ✅ Never store keyfile on the same encrypted disk
* ✅ Encrypt swap partitions
* ✅ Avoid exposing passwords in shell history
* ✅ Always test on VM before production
* ✅ Use LUKS2 for best security
* ✅ Use AES-XTS-512 cipher

---

✅ End of LUKS Ultimate English Cheat Sheet

```

