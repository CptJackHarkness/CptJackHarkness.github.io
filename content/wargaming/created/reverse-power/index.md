---
title: "Reverse Power — File Carving & Magic Bytes"
date: 2026-06-03
description: "A forensics challenge involving corrupted file signatures, hex editing, EXIF password extraction, and password-protected ZIP archives."
tags:
    - Forensics
    - File Carving
    - Hex Editing
    - EXIF
    - Magic Bytes
---

## Overview

"Reverse Power" is a practical forensics exercise focused on low-level data manipulation, covering file carving and file format analysis. It explores how file signatures (magic bytes) work and how to recover files when those signatures are deliberately corrupted.

---

## Techniques Covered

- **File Signatures (Magic Bytes):** Most file types include unique identifiers in their first bytes. ZIPs begin with `50 4B 03 04`, JPEGs with `FF D8`. The challenge requires identifying and restoring corrupted signatures.
- **Hex Editing:** Directly inspecting and altering raw binary content of a file.
- **EXIF Metadata:** Extracting passwords hidden in image metadata fields.
- **Password-Protected Archives:** Using recovered passwords to unlock ZIP files.

---

## Challenge Setup

1. A `flag.txt` containing `flaG_fSt_{R3v3rse_M451C}` was compressed and password-protected into `important.zip`
2. The ZIP password (`z1p1nz1p`) was hidden in the EXIF Comment field of `broken.jpg`
3. The first 2 bytes of `broken.jpg` were corrupted to `RE`, breaking its JPEG signature
4. All files were packaged into `Serious_Business.pdf` — actually a ZIP with its first 8 bytes overwritten to look like a PDF 1.4 header

---

## Solution Walkthrough

### Step 1 — Identify & Repair the Main ZIP
```bash
file Serious_Business.pdf
```
The `file` command reveals it is actually a ZIP. Extract it:
```bash
unzip Serious_Business.pdf
```

---

### Step 2 — Repair the Corrupted Image
Inside the extracted folders, locate `broken.jpg` and `important.zip`. The image won't open because its signature is corrupted. Open it in a hex editor and replace the first 2 bytes:

| Offset | Corrupted | Correct |
|--------|-----------|---------|
| 0x00   | `52 45` (RE) | `FF D8` (JPEG) |

---

### Step 3 — Extract the Password and Flag
With the image repaired, read its EXIF Comment field:
```bash
exiftool broken/folder2/broken.jpg | grep -i "comment"
```
**Password found:** `z1p1nz1p`

Use it to unlock the protected archive:
```bash
unzip broken/folder4/important.zip
```

---

## What I Learned

File extensions are just labels — the actual file type is determined by its internal signature. Attackers and defenders alike need to understand magic bytes: attackers use them to disguise malicious files, and analysts use them to correctly identify what a file actually is regardless of what it claims to be.

---

## Flag

`fLaG_fSt_{R3v3rse_M451C}`
