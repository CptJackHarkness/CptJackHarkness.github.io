---
title: "Emergency Notes — Double XOR Cryptography"
date: 2026-06-03
description: "A cryptography challenge focused on the XOR operation and its reversibility, requiring players to test key combinations to decrypt a double-XOR-encrypted backup."
---

## Overview

"Emergency Notes" is a practical cryptography exercise focused on the XOR (Exclusive OR) logical operation. XOR compares bits and has two fundamental properties:
- **Self-inverse:** `A XOR A = 0`
- **Identity:** `A XOR 0 = A`

When a readable message has XOR applied with a key, it becomes unreadable ciphertext. Applying XOR again with the same key perfectly recovers the original message.

The scenario: a sysadmin created an emergency backup and protected it with a homemade XOR scheme. The player receives a notes file containing the encrypted content in hexadecimal format and three possible passphrases. Only two of the three were used — the goal is to find the correct combination.

---

## Challenge Creation

Built entirely in CyberChef:

1. Started with the original flag + ASCII art: `fLaG_fSt{N1c0lay_emergency_backup}`
2. Applied XOR twice using two distinct passphrases (UTF-8 format):
   - `xor is not backup`
   - `do not use in production`
3. Converted the result to hexadecimal (to make it safe to store as text)
4. Created `notes.txt` containing the hex output and **three** candidate phrases — including `emergency backup` as a red herring

---

## Solution Walkthrough

### Step 1 — Open CyberChef
Copy the long hex string from `notes.txt` and paste it into the CyberChef Input field.

### Step 2 — Reverse the Hex
Add a **From Hex** operation to the recipe to convert back to binary.

### Step 3 — Test XOR Combinations
With three candidate phrases and only two used, there are exactly 3 possible combinations. Add two XOR operations to the recipe, setting the Key format to UTF-8.

The correct combination:
- **XOR 1:** `xor is not backup`
- **XOR 2:** `do not use in production`

### Step 4 — Capture the Flag
With the two correct keys applied, XOR's reversibility cancels the cipher and the Output reveals the flag.

**Alternatively via terminal:**
```python
python3 -c '
import functools, operator
data = bytes.fromhex("PASTE_HEX_HERE")
keys = [b"xor is not backup", b"do not use in production"]
result = data
for key in keys:
    result = bytes(b ^ key[i % len(key)] for i, b in enumerate(result))
print(result.decode())
'
```

---

## What I Learned

XOR is not a secure encryption scheme — it is trivially reversible if the key is known or guessable. This challenge reinforces why symmetric encryption should never be implemented with simple XOR against static passphrases, especially when both the ciphertext and candidate keys are available to the attacker.

---

## Flag

`fLaG_fSt{N1c0lay_emergency_backup}`
