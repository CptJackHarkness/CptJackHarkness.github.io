---
title: "The True Owner — LFI, Base64 & Cookie Tampering"
date: 2026-06-03
description: "A web challenge chaining OWASP Top 10 vulnerabilities: information disclosure via robots.txt, Local File Inclusion, Base64-encoded credentials, and cookie tampering for privilege escalation."
tags:
    - Web
    - LFI
    - OWASP
    - Cookie Tampering
    - Base64
    - Path Traversal
---

## Overview

"The True Owner" explores a chain of web vulnerabilities based on the OWASP Top 10 (2025). The challenge requires progressing through multiple attack layers in sequence — a realistic reflection of how real-world web attacks chain smaller weaknesses into full compromise.

---

## Vulnerabilities Covered

- **Information Disclosure (A02:2025 — Security Misconfiguration):** Mapping `robots.txt` directives to discover intentionally hidden paths.
- **LFI / Path Traversal (A01:2025 — Broken Access Control):** Exploiting inadequate URL parameter validation to read arbitrary files from the server.
- **Data Encoding:** Identifying and reversing Base64-encoded credentials.
- **Cookie Tampering (A07:2025 — Authentication Failures):** Escalating privileges by manipulating a client-side role cookie that the server blindly trusts.

---

## Challenge Setup

- A PHP/Apache container with `/secrets.txt` in the root, containing credentials in Base64: `Guylherm:YXRlcXVpX293bmVyCg==`
- `robots.txt` with `Disallow: /notes.txt`
- `notes.txt` pointing to `/secrets.txt`
- `wargame.php` directly passing `?file=` to `@include()` — classic LFI
- Login validation performed entirely client-side; after login, the server sets a cookie `role=user`. Showing `role=admin` reveals the flag.

---

## Solution Walkthrough

### Step 1 — Add Host Entry
```bash
echo "172.30.2.32 atequi.ctf" | sudo tee -a /etc/hosts
```

### Step 2 — Reconnaissance via robots.txt
```
http://atequi.ctf/robots.txt
```
Reveals `/notes.txt`. Reading it discloses the existence of `/secrets.txt` and hints at the LFI vulnerability via the `file` parameter.

### Step 3 — Local File Inclusion (Path Traversal)
In the "Info (Wargame)" page, exploit the vulnerable `?file=` parameter:
```
wargame.php?file=../../../../../../secrets.txt
```
**Server returns:** `Guylherm:YXRlcXVpX293bmVyCg==`

### Step 4 — Decode Base64 & Login
```bash
echo "YXRlcXVpX293bmVyCg==" | base64 -d
# Output: atequi_owner
```
Use credentials `Guylherm:atequi_owner` at `login.php`.

### Step 5 — Cookie Tampering
After login, open browser Developer Tools (`F12`) → Application/Storage → Cookies. Change the `role` cookie value from `user` to `admin`.

### Step 6 — Flag
Refresh the page (`F5`). The server trusts the forged cookie and renders the admin area with the flag.

---

## What I Learned

This challenge demonstrates that trusting client-controlled data — whether URL parameters, form fields, or cookies — is never safe. The `role` cookie should have been validated server-side against a session record, and the `?file=` parameter should have been sanitised to prevent directory traversal. These are foundational web security mistakes that still appear in real applications.

---

## Flag

`fLaG_fSt_{I_0Wn_p@Ul_fIsHeR}`
