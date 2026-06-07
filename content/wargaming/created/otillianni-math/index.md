---
title: "Otillianni Math — Prime Number Indexing"
date: 2026-06-03
description: "A data obfuscation challenge where a flag is hidden inside 10,000 random characters by placing each flag character at prime number indices."
tags:
    - Scripting
    - Python
    - Mathematics
    - Obfuscation
---

## Overview

"Otillianni Math" focuses on mathematical obfuscation. The challenge hides a readable flag by diluting it inside a large block of random characters (noise). To both hide and index the message, a specific mathematical sequence is used: **prime numbers**. The characters composing the flag are placed exclusively at indices that correspond to exact prime numbers within a 10,000-character string.

Solving the challenge requires writing an automation script capable of computing prime numbers and reading only the correct positions from the extracted text to reconstruct the original string.

---

## Challenge Creation

A Python script:
1. Generated a 10,000-character base string of random alphanumeric characters
2. Defined an `is_prime(n)` function that iterates only up to the square root of `n` for efficiency
3. At each index corresponding to a prime number, overwrote the random character with the next character of the flag: `fLaG_fSt_{@t3qU1_m@tH}`
4. After the flag was fully written, continued overwriting remaining prime indices with new random characters (to prevent pattern detection by length)

---

## Solution Walkthrough

### Step 1 — Read the File
```python
with open("otilianni.txt", "r") as f:
    content = f.read()
```

### Step 2 — Implement the Prime Check Function
```python
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True
```

### Step 3 — Extract Characters at Prime Indices
Iterate over the string, collect characters at prime positions, and stop as soon as the closing `}` is found (to avoid capturing the random noise added after the flag):
```python
flag = ""
for i in range(len(content)):
    if is_prime(i):
        flag += content[i]
        if flag[-1] == "}":
            break

print(flag)
```

---

## What I Learned

Prime numbers have a property that makes them useful for data indexing — they are unpredictable to the naked eye and distributed across large ranges. This challenge reinforced the importance of recognising patterns in data and the value of scripting as an analytical tool. Without automation, manually checking 10,000 character positions would be infeasible.

---

## Flag

`fLaG_fSt_{@t3qU1_m@tH}`
