---
title: "Roda do Hamster — Layered Encoding Decoder"
date: 2026-06-03
description: "A challenge solved by scripting a recursive decoder that automatically strips multiple layers of Base16/32/64/85 and Ascii85 encoding until plain text is revealed."
---

## Overview

"Roda do Hamster" (Hamster Wheel) presented a file encoded in multiple successive layers of different encoding schemes. The `README.txt` described the approach required: since there were no passwords or symmetric keys, the solution was pure automated decoding — cycling through encoding algorithms until natural language emerged.

---

## Approach

A Python script (`quebra.py`) was developed and run against `roda.txt` following this logic:

1. **Cyclic Recognition:** The script tests the content against native decoding algorithms in sequence: Ascii85, Base85, Base64, Base32, Base16.
2. **Sanitisation & Padding:** For encodings requiring strict block sizes (Base64, Base32), the algorithm dynamically calculates and injects missing padding (`=`) to prevent truncated payload errors.
3. **Decoding Validation:** If the output generates bytes that translate to clean UTF-8 text, the engine marks that layer as successfully broken.
4. **Stopping Condition:** The loop iterates recursively until all encoding layers are stripped and the result is natural language where the mathematical conversions fail.

---

## Script Logic (quebra.py)

```python
import base64, binascii

def try_decode(data):
    encodings = [
        ("Base64", lambda d: base64.b64decode(d + "=" * (-len(d) % 4))),
        ("Base32", lambda d: base64.b32decode(d + "=" * (-len(d) % 8), casefold=True)),
        ("Base16", lambda d: base64.b16decode(d, casefold=True)),
        ("Base85", lambda d: base64.b85decode(d)),
        ("Ascii85", lambda d: base64.a85decode(d)),
    ]
    for name, fn in encodings:
        try:
            result = fn(data.strip())
            decoded = result.decode("utf-8")
            print(f"[+] Decoded layer: {name}")
            return decoded
        except Exception:
            continue
    return None

with open("roda.txt", "r") as f:
    content = f.read().strip()

while True:
    result = try_decode(content)
    if result is None:
        print(f"\n[*] Final result:\n{content}")
        break
    content = result
```

---

## What I Learned

When an encoding problem has no cryptographic key, scripted brute-force decoding across common encoding schemes is the right approach. The key insight here was recognising that Base64, Base32, and similar schemes are **encodings** (not encryption) and are trivially reversible — the challenge was just in chaining them automatically. Padding handling is a common pitfall.

---

## Flag

`CTF{5cr1pt1ng_m45t3r_p0rt0_2026}`
