---
title: "LaCasa — FTP, Steganography & Privilege Escalation"
date: 2026-06-03
description: "A Money Heist-themed challenge chaining anonymous FTP access, image steganography, SSH lateral movement, and vim-based root privilege escalation."
---

## Overview

"LaCasa" is a Money Heist (La Casa de Papel)-themed challenge involving multiple attack stages: anonymous FTP access, credential discovery, image steganography, lateral movement via SSH, and privilege escalation using GTFOBins.

---

## Solution Walkthrough

### Step 1 — Reconnaissance
```bash
nmap <target-ip>
```
Identified open ports including FTP and HTTP. Visited the website — `robots.txt` yielded nothing useful. The HTML source contained placeholder credentials that did not work on the login page.

---

### Step 2 — Anonymous FTP Access
Anonymous FTP login was enabled. Navigated to the accessible `pub` folder and found a file containing a credential hint: `toquio:BellaCiao`.

---

### Step 3 — FTP Login as Tóquio
Used the recovered credentials to log in as `toquio` and accessed the previously locked folder. Found 3 files — one containing the first flag.

**Flag 1:** `FLAG{T0qu10_F0und_Th3_W4y}`

---

### Step 4 — Image Steganography
Analysed image files from the FTP server. One contained hidden data — extracted using `zsteg`:
```bash
zsteg <image>
```
Output: a Base64-encoded hash. Decoded it to reveal the name **Silene Oliveira** — the actress who plays the character Tóquio in the series.

---

### Step 5 — Web Login & Further Enumeration
Used "Silene Oliveira" as a credential to log into the website. Gained access to an authenticated area containing 4 files:
- Two files with a **latitude** and **longitude**
- A **candidates list**
- A **capitals list**

The coordinates (via Google Maps) pointed to **Berlin**. Cross-referencing with the candidates list, identified the actor playing the character "Berlim": **defonollosaandres**.

---

### Step 6 — SSH Access as Berlim
```bash
ssh Berlim@<target-ip>
```
Found a flag hidden in the shell history:

**Flag 2:** `FLAG{B3rl1m_H1st0ry_Unl0ck3d}`

---

### Step 7 — Credential Discovery & Lateral Movement
Continued exploring the home folder. Decoded an encoded file to reveal new credentials: `professor:V@tic4n0`. Switched user:
```bash
su professor
```

---

### Step 8 — Root Privilege Escalation via vim
Checked sudo permissions:
```bash
sudo -l
# professor can run vim as root without a password
```
Used [GTFOBins](https://gtfobins.github.io/gtfobins/vim/) to spawn a root shell:
```bash
sudo vim -c ':!/bin/bash'
```
Accessed the root account and captured the third flag.

**Flag 3:** `FLAG{Pr0f3ss0r_M4st3rm1nd_CTF_COMPLETE}`

---

### Step 9 — Final Flag
Reviewed previously obtained files. Found the fourth flag inside `capitais.txt`.

**Flag 4:** `FLAG{Porto_B3rc0_N4c4o}`

---

## Flags

| # | Flag |
|---|------|
| 1 | `FLAG{T0qu10_F0und_Th3_W4y}` |
| 2 | `FLAG{B3rl1m_H1st0ry_Unl0ck3d}` |
| 3 | `FLAG{Pr0f3ss0r_M4st3rm1nd_CTF_COMPLETE}` |
| 4 | `FLAG{Porto_B3rc0_N4c4o}` |
