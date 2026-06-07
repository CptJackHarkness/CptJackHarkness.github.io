---
title: "Reactor — Binary Analysis & XOR Decoding"
date: 2026-06-03
description: "A reverse engineering challenge involving string extraction from a binary, hex-to-ASCII decoding, XOR binary-to-decimal conversion, and a two-stage access code challenge."
---

## Overview

"Reactor" is a reverse engineering challenge built around a binary executable that requests two sequential access codes. The solution requires extracting strings from the binary, decoding values in hex and binary, and applying XOR to uncover the second code.

---

## Solution Walkthrough

### Step 1 — Run the Binary
```bash
./reactor
```
The program prompts for an access code.

---

### Step 2 — Extract Strings from the Binary
```bash
strings reactor
```
Among the extracted strings, a value labelled `primary_sequence` was found encoded in hexadecimal. A `backup_sequence` also appeared at the end.

**Hex code found:** `A73C`

---

### Step 3 — First Access Code
Entered `A73C` as the first access code. The program accepted it and moved to the second prompt, generating a dump file.

---

### Step 4 — Analyse the Dump
Inspected the dump file. It revealed:
- An XOR operation using key `0x32`
- A number encoded in **binary**

Verified the CyberChef hint embedded in the binary fragment — it confirmed the XOR operation.

---

### Step 5 — Decode the Second Code
1. Converted the binary number to decimal
2. Applied XOR with `0x32` to each byte
3. Decoded the result to ASCII

**Second code found:** `C3N4S`

---

### Step 6 — Flag
Entered `C3N4S` as the second code. The program revealed the flag.

---

## What I Learned

Binary analysis with `strings` is often the first step in reverse engineering — many compiled programs contain readable artefacts that reveal significant information about their logic. Combined with CyberChef for XOR and encoding operations, simple RE challenges can often be solved without a full disassembler.

---

## Flag

`FLAG{r3ac70r_c0r3_s74b1l1z3d}`
