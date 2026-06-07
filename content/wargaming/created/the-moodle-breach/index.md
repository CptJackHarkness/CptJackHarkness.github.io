---
title: "The Moodle Breach - Multi-Vector Forensics"
date: 2026-06-03
description: "A miscellaneous challenge combining HTML source analysis, XOR decryption, EXIF metadata extraction, LSB steganography, Base64 decoding, and visual forensics to recover 6 flags."
---

## Overview

"The Moodle Breach" is a miscellaneous challenge requiring analysis of multiple artefacts — HTML documents, images, and text files - to extract six distinct flags. It covers a broad range of techniques in one scenario.

---

## Techniques Covered

1. HTML source analysis + XOR decryption
2. EXIF metadata extraction + hex decoding
3. LSB steganography (zsteg)
4. HTML meta tag analysis + hex decoding
5. Base64 decoding
6. Visual forensics (screenshot analysis / OCR)

---

## Solution Walkthrough

### Flag 1 - HTML Comment + XOR (`incident_report.html`)

Open the file and locate two comments: one at the top with `schema: 4d` (the XOR key) and one at the bottom with `tok=2b012c...` (the encrypted data).

**Verify the key:** XOR the ASCII values of `f` and `l` against the first 2 bytes of the token — if the result matches `4d`, the assumption is confirmed.

**In CyberChef:** From Hex → XOR (key: `4d`, mode: Hex)

**Via terminal:**
```python
python3 -c '
key = 0x4d
tok = "2b012c0a122b1e39123625392021122e7d207e23391279397e2e30"
flag = bytes(b ^ key for b in bytes.fromhex(tok)).decode("utf-8")
print(flag)
'
```
**Result:** `fLaG_fSt_{html_c0m3nt_4t3c}`

---

### Flag 2 - EXIF Metadata + Hex (`captura_03_moodle_admin.jpg`)

```bash
exiftool captura_03_moodle_admin.jpg | grep -i description
# Returns: 664c61475f6653745f7b337831665f6d3030646c335f34646d316e7d

echo "664c61475f6653745f7b337831665f6d3030646c335f34646d316e7d" | xxd -r -p
```
**Result:** `fLaG_fSt_{3x1f_m00dl3_4dm1n}`

---

### Flag 3 - LSB Steganography (`captura_02_burpsuite_intercept.png`)

The `integrity_hashes.txt` hints at "lower bit planes of RGB channels".
```bash
zsteg captura_02_burpsuite_intercept.png
```
Alternatively, use StegOnline online — upload the file, click **Extract Files/Data**, and activate the LSB scan (least significant bits).

**Result:** `fLaG_fSt_{l5b_1nt3rc3pt3d}`

---

### Flag 4 - Meta Tag + Hex (`index.html`)

Inspect the HTML source and locate an anomalous meta tag:
```html
<meta name="booking-ref" content="664c61475f6653745f7b6d3374345f347433635f68316464336e7d">
<meta name="data-encoding" content="hex/UTF-8">
```
Decode the hex string via CyberChef (From Hex) or terminal.

**Result:** `fLaG_fSt_{m3t4_4t3c_h1dd3n}`

---

### Flag 5 - Base64 (`wargaming_notes.txt`)

The file contains a WiFi note with a password ending in `==` - a clear Base64 padding indicator.
```bash
echo "ZkxhR19mU3Rfe2I0czM2NF93NHJnNG0zfQ==" | base64 -d
```
**Result:** `fLaG_fSt_{b4s364_w4rg4m3}`

---

### Flag 6 - Visual Analysis (`captura_06_php_source.png`)

Inspect the screenshot carefully. The value of the `$CFG->sessioncookie` PHP variable has been replaced with the flag. Read it directly from the image or use an OCR tool.

**Result:** `fLaG_fSt_{php_cfg_s3ss10n_l34k}`

---

## Flags

| # | Technique | Flag |
|---|-----------|------|
| 1 | HTML Comment + XOR | `fLaG_fSt_{html_c0m3nt_4t3c}` |
| 2 | EXIF + Hex | `fLaG_fSt_{3x1f_m00dl3_4dm1n}` |
| 3 | LSB Steganography | `fLaG_fSt_{l5b_1nt3rc3pt3d}` |
| 4 | Meta Tag + Hex | `fLaG_fSt_{m3t4_4t3c_h1dd3n}` |
| 5 | Base64 | `fLaG_fSt_{b4s364_w4rg4m3}` |
| 6 | Visual Forensics | `fLaG_fSt_{php_cfg_s3ss10n_l34k}` |
