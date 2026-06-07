---
title: "Handsome Peter — Steganography Challenge"
date: 2026-06-03
description: "A multi-technique steganography challenge covering EXIF metadata, file appending, LSB, DCT, colour palette manipulation, and audio steganography."
---

## Overview

"Handsome Peter" is a practical journey through steganography, covering 6 fundamental data-hiding techniques. Steganography is the practice of concealing information inside other data (media files, metadata, etc.) in an imperceptible way. Unlike cryptography — which protects data by making it unreadable — steganography hides the very existence of the message.

The objective is to identify and extract 6 flags hidden across different files using 6 distinct steganography techniques.

---

## Techniques Covered

- **EXIF Metadata:** Hides information in a file's hidden fields (comments, author, date, location). The image pixels remain untouched, making the concealment completely invisible.
- **Append/Trailer:** Inserts arbitrary data after the end-of-file marker (EOF). In JPEG, for example, the marker `FF D9` marks the end of the image. Everything after it is ignored by viewers but remains in the file and can be extracted.
- **LSB (Least Significant Bit):** Alters the last bit of the RGB colour values of pixels. Since the human eye is insensitive to such subtle colour changes, the information is imperceptible. Works best with uncompressed formats like PNG and BMP.
- **DCT (Discrete Cosine Transform):** Exploits the mathematical frequencies used in JPEG compression. Data is inserted into DCT coefficients during the transform, protected by the compression algorithm's own structure.
- **Colour Palette:** In indexed formats (GIF, PNG-8), creates imperceptible duplicate colours and manipulates palette order to encode information.
- **Audio Steganography:** Hides data by modifying bits in audio samples (WAV), altering amplitude or frequency so subtly that the human ear detects no distortion.

---

## Challenge Creation

**Metadata:**
```bash
exiftool -Comment="fLaG_fSt_{M3tAdAtA_sTeG_iS_fUn}" image02.jpg
```

**Append/Trailer:**
```bash
echo fLaG_fSt_{App3nD_l4y3r5_sTeG} >> image05.jpg
cat image05.jpg password_1.zip >> image05final.jpg
```

**LSB:**
```bash
stegano-lsb hide -i image07.png -m "fLaG_fSt_{LsB_sTeG_iS_SiGnIficaNt}" -o image07.png
```

**DCT:**
```bash
echo "fLaG_fSt_{DcT_sTeG_iS_dIsCrEte}\nP3t3rSt3g0M4st3r" | steghide embed -ef - -cf image09.jpg -p 'H4nds0meP3ter!#$'
```

**Colour Palette:**
```bash
gifshuffle -m "fLaG_fSt_{PaLeTtE_oRdEr_mAtTeRs_sTeG}" image14.gif image14final.gif
```

**Audio:**
```bash
echo "fLaG_fSt_{AuDi0_AmPlitUd3_M4st3r_sTeG}" | steghide embed -ef - -cf music02.wav -p 'P3t3rSt3g0M4st3r'
```

---

## Solution Walkthrough

### Step 1 — EXIF Metadata (image02.jpg)
Inspect the image metadata using exiftool. The flag will be in the "Comment" field.
```bash
exiftool image02.jpg
# Alternative:
strings image02.jpg | grep fLaG_fSt_
```
**Result:** `fLaG_fSt_{M3tAdAtA_sTeG_iS_fUn}`

---

### Step 2 — Append/Trailer (image05.jpg)
Use `tail` to view the flag and `binwalk` to extract appended data.
```bash
tail -n 5 image05.jpg
binwalk -e image05.jpg
cd _image05.jpg.extracted && ls -la && cat password_1.txt
```
**Result:** `fLaG_fSt{App3nD_l4y3r5_sTeG}`  
**Password found:** `H4nds0meP3ter!#$`

---

### Step 3 — LSB (image07.png)
Use `zsteg` to automatically analyse LSB layers.
```bash
zsteg -a image07.png
# More targeted:
zsteg -a image07.png 2>/dev/null | grep -i fLaG_fSt_
```
**Result:** `fLaG_fSt_{LsB_sTeG_iS_SiGnIficaNt}`

---

### Step 4 — DCT (image09.jpg)
Extract data hidden in DCT frequencies with steghide, using the password found in Step 2.
```bash
steghide extract -sf image09.jpg -xf output.txt -p 'H4nds0meP3ter!#$' && cat output.txt
```
**Result:** `fLaG_fSt_{DcT_sTeG_iS_dIsCrEte}`  
**Password found:** `P3t3rSt3g0M4st3r`

---

### Step 5 — Colour Palette (image14.gif)
Use `gifshuffle` to reveal data hidden in the colour palette.
```bash
gifshuffle image14.gif
```
**Result:** `fLaG_fSt_{PaLeTtE_oRdEr_mAtTeRs_sTeG}`

---

### Step 6 — Audio Steganography (music02.wav)
Extract data from the audio file with steghide, using the password found in Step 4.
```bash
steghide extract -sf music02.wav -xf audio_output.txt -p 'P3t3rSt3g0M4st3r'
cat audio_output.txt
```
**Result:** `fLaG_fSt_{AuDi0_AmPlitUd3_M4st3r_sTeG}`

---

## Flags

| # | Technique | Flag |
|---|-----------|------|
| 1 | EXIF Metadata | `fLaG_fSt_{M3tAdAtA_sTeG_iS_fUn}` |
| 2 | Append/Trailer | `fLaG_fSt_{App3nD_l4y3r5_sTeG}` |
| 3 | LSB | `fLaG_fSt_{LsB_sTeG_iS_SiGnIficaNt}` |
| 4 | DCT | `fLaG_fSt_{DcT_sTeG_iS_dIsCrEte}` |
| 5 | Colour Palette | `fLaG_fSt_{PaLeTtE_oRdEr_mAtTeRs_sTeG}` |
| 6 | Audio | `fLaG_fSt_{AuDi0_AmPlitUd3_M4st3r_sTeG}` |
