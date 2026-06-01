---
title: "Quick Writeup: Network Reconnaissance Analysis with Nmap"
image: cover.png
description: "How to identify open ports and vulnerable services on a target host."
date: 2026-05-16
tags:
    - Nmap
    - Networks
    - Lab
draft: false
---

## Lab Objective
The objective of this exercise was to perform a reconnaissance scan (*recon*) on a test host to map the attack surface and identify potential entry vectors.

---

## 1. Command Execution
I used Kali Linux to run an aggressive scan, extracting service versions and the target's operating system:

```bash
nmap -sV -sC -O -F 192.168.1.50
```

---

## 2. Results

| Port | State | Service | Version Detected | Security Impact |
| :--- | :--- | :--- | :--- | :--- |
| **22/TCP** | Open | SSH | OpenSSH 8.2p1 | Secure (if using keys instead of passwords) |
| **80/TCP** | Open | HTTP | Apache httpd 2.4.41 | Potential vector if public exploits exist |
| **445/TCP**| Open | SMB | Samba 4.X | **Critical:** Common target for lateral movement |

---

## 3. Conclusion

The exposed 445 (SMB) port represents the highest risk in this scenario. Immediate mitigation recommendations:

1. Isolate port 445 using perimeter Firewall rules.
2. Ensure SMBv1 protocol is completely disabled, forcing the use of encrypted SMBv2/v3.