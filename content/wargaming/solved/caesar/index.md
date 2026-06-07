---
title: "Caesar — Hash Cracking & ROT8000"
date: 2026-06-03
description: "A two-part challenge combining SHA256 hash cracking, custom XOR-based decryption, and ROT8000 Unicode rotation to decode a message disguised as East Asian ideograms."
tags:
    - Cryptography
    - Hash Cracking
    - ROT8000
    - XOR
    - CyberChef
---

## Overview

"Caesar" is a two-part cryptographic challenge. The first part requires cracking a SHA256 hash to obtain a decryption key. The second part involves decoding a message that has been rotated using ROT8000 — a Unicode rotation that transforms ASCII characters into characters resembling East Asian ideograms.

---

## Solution Walkthrough

### Part 1 — Hash Cracking

The first message provided this hash:
```
6ec4dca2bcc10d9f68fbd938aefd9e8daff46c1674e09501e2520dd2c82b1e6f
```

Identified the hash type with `hashid` — the 64-character hex string is consistent with SHA256. Used [CrackStation](https://crackstation.net) to look it up in its database.

**Hash cracked:** `veni`

---

### Part 2 — Decrypting the Encrypted File

Used `veni` as the key to decrypt `bau.enc` with the provided `chave.py` script. The decrypted message revealed the flag format used by the challenge creators:
```
C4es4r ... w4s ... [secret]
```
This established the expected flag structure: `C4es4r_w4s_[secret]`

---

### Part 3 — Decoding the Final Message

The second message contained a riddle hinting at a "rotation" that makes Western letters look like "ideograms from a distant dynasty". The final encoded message was:
```
籫簼籽类簽粂簼籭籨籁粂籨籾簾
```

Tested in order:
- **ROT13** — no useful output
- **ROT47** — no change
- **ROT8000** — success. ROT8000 is a Unicode rotation capable of transforming ASCII characters into characters resembling ideograms.

**Result:** `b3tr4y3d_8y_u5`

Combining with the format discovered earlier:

**Final flag:** `C4es4r_w4s_b3tr4y3d_8y_u5`

---

## What I Learned

ROT8000 is a lesser-known rotation cipher that operates on Unicode code points, not just ASCII. It is specifically designed to make text look like CJK (Chinese/Japanese/Korean) characters — making it visually unrecognisable to anyone unfamiliar with it. This challenge reinforced the importance of exploring the full range of encoding and rotation schemes, not just the classic ROT13 and ROT47.

---

## Flag

`C4es4r_w4s_b3tr4y3d_8y_u5`
