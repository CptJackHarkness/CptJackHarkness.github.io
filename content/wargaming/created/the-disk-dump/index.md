---
title: "The Disk Dump — FAT12 Forensics & XOR Decryption"
date: 2026-06-03
description: "A forensic imaging challenge involving FAT12 filesystem analysis, recovery of deleted files, a decoy rabbit hole, and XOR-based flag decryption."
tags:
    - Forensics
    - File Carving
    - FAT12
    - XOR
    - Disk Imaging
    - Autopsy
---

## Overview

"The Disk Dump" focuses on forensic analysis of FAT12 filesystems and fundamental cryptography. It explores how FAT handles file deletion: when a file is deleted, the data remains intact on disk — only the first byte of the filename in the directory entry is replaced with `0xE5`, marking the space as available.

The challenge also includes a **decoy flag** (rabbit hole) designed to mislead players.

---

## Challenge Setup

A FAT12 disk image (`the_disk_dump.img`) was created and mounted. Four files were written and then forensically deleted (first byte replaced with `0xE5`):

| File | Deleted Byte |
|------|-------------|
| `NOTE.TXT` | `4E` (N) → `E5` |
| `CIPHER.CFG` | `43` (C) → `E5` |
| `PASSWD.BAK` | `50` (P) → `E5` |
| `SYSLOG.BAK` | `53` (S) → `E5` |

`SYSLOG.BAK` contains a Base64-encoded fake flag to serve as the rabbit hole. `NOTE.TXT` contains encrypted data. `CIPHER.CFG` contains the XOR key.

---

## Solution Walkthrough

### Step 1 — Enumerate & Extract Deleted Files

**Via command line (The Sleuth Kit):**
```bash
fls the_disk_dump.img
# Lists deleted files marked with *

icat the_disk_dump.img 2 > NOTE.TXT
icat the_disk_dump.img 7 > PASSWD.BAK
icat the_disk_dump.img 8 > SYSLOG.BAK
icat the_disk_dump.img 9 > CIPHER.CFG
```

**Via GUI (Autopsy):** Create a new case, import the disk image, and browse to **Deleted Files → All** to see the four recovered files.

---

### Step 2 — Identify the Rabbit Hole
Inspecting `SYSLOG.BAK` reveals a log line with an encoded string. Decoding it produces a flag — but the format reveals it is **fake**. Do not stop here.
```bash
icat the_disk_dump.img 9
```

---

### Step 3 — Collect the Real Key and Data
- `CIPHER.CFG` contains: `enc_key=0x3F`
- `NOTE.TXT` contains: `enc_data=59735E7860596C4B60445B0E4C54605C0B4D490E515842`

---

### Step 4 — XOR Decryption

**Via CyberChef:**
1. Input: the hex string from `NOTE.TXT`
2. Add operation: **From Hex** (delimiter: Auto)
3. Add operation: **XOR** (Key: `3F`, Key format: HEX, Scheme: Standard)

**Via terminal:**
```python
python3 -c '
enc_hex = "59735E7860596C4B60445B0E4C54605C0B4D490E515842"
flag = bytes(b ^ 0x3F for b in bytes.fromhex(enc_hex)).decode()
print(flag)
'
```

---

## What I Learned

Deleted does not mean gone. In FAT filesystems, deleted file data persists until its clusters are overwritten by new data. This is fundamental to digital forensics — and also a reminder that securely wiping data requires overwriting, not just deleting. The rabbit hole in this challenge also reinforced the importance of not stopping at the first flag-looking result.

---

## Flag

`fLaG_fSt_{d1sk_c4rv1ng}`
