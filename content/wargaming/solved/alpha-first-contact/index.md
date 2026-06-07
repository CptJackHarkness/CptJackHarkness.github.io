---
title: "Alpha — First Contact"
date: 2026-06-03
description: "The introductory wargaming challenge requiring a simple /etc/hosts file edit to resolve a custom domain."
tags:
    - Networking
    - DNS
    - Linux
---

## Overview

The first challenge of the wargaming competition served as a warm-up, requiring only a single configuration step to resolve a custom CTF domain.

---

## Solution

Add the target machine's IP to the local `/etc/hosts` file so the custom domain resolves correctly:
```bash
echo "<target-ip> alpha.ctf" | sudo tee -a /etc/hosts
```

With the domain resolved, the challenge page became accessible and the flag was visible.

---

## Flag

`CISEG{alpha_first_contact}`
