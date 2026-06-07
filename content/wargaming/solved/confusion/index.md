---
title: "Confusion - Multi-Layer Steganography with Movie Clues"
date: 2026-06-03
description: "A three-part steganography challenge using movie-themed passphrases, steghide extraction, Base64/Base32 decoding, and nested ZIP archives to reconstruct a split flag."
---

## Overview

"Confusion" presented a split flag divided across three files, each requiring a different passphrase to extract hidden content. The challenge used movie-themed clues (specifically referencing *Inception*) and a Disney easter egg to guide passphrase discovery. All passwords had to be entered in uppercase.

---

## Solution Walkthrough

### Step 1 - Read the README
`readme.txt` stated:
- All passwords must be in uppercase
- The flag is split across 3 files

---

### Part 1 - First Image (100.jpeg)

Analysed the EXIF metadata of `100.jpeg`:
```bash
exiftool 100.jpeg
```
A metadata hint suggested the passphrase could be **TOTEM** or **ILUSION**.

Tested both with steghide:
```bash
steghide extract -sf 100.jpeg -p TOTEM
```
Successfully extracted a file `?.txt`.

The extracted file contained a Base64-encoded hash. Decoded it in CyberChef (From Base64):

**Flag Part 1:** `FLAG{s3gr3d0_`

---

### Part 2 - Second Image

EXIF metadata on the second image hinted that the passphrase was the film referenced by the images - **INCEPTION**.

```bash
steghide extract -sf 200.jpeg -p INCEPTION
```
Extracted a file containing a Base32-encoded hash. Decoded it in CyberChef (From Base32):

**Flag Part 2:** `d1v1d1d0_f01_3sc0nd1d0_`

---

### Part 3 - Third File ("A113")

The file was named `A113`, a well-known Disney easter egg appearing across many films. Used **DISNEY** as the passphrase to extract the hidden content:

```bash
steghide extract -sf A113 -p DISNEY
```
Extracted a ZIP file. Unzipped it:
```bash
unzip <extracted_file>
```
The second file's clue had mentioned **Limbo** as the solution for the final challenge (a reference to *Inception*'s concept of dream layers). Used **LIMBO** to extract the final hidden file:

```bash
steghide extract -sf 300.jpeg -p LIMBO
```
Decoded the extracted hash to obtain the last flag fragment:

**Flag Part 3:** `c0m_suc3ss0}`

---

### Assembling the Flag
Joined the three parts:

`FLAG{s3gr3d0_` + `d1v1d1d0_f01_3sc0nd1d0_` + `c0m_suc3ss0}`

---

## What I Learned

This challenge reinforced that steganography passphrases are often hidden in plain sight with metadata, cultural references, or thematic clues embedded in the challenge's narrative. Methodical metadata inspection with `exiftool` and familiarity with common film references (like the A113 easter egg) are genuinely useful skills in CTF steganography.

---

## Flag

`FLAG{s3gr3d0_d1v1d1d0_f01_3sc0nd1d0_c0m_suc3ss0}`
