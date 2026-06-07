---
title: "Football Leaks — Web Vulnerabilities Chain"
date: 2026-06-03
description: "A web challenge exploiting client-side authentication bypass, IDOR, and SQL injection to extract three flags from a football data leak portal."
tags:
    - Web
    - IDOR
    - SQL Injection
    - Authentication Bypass
    - OWASP
---

## Overview

"Football Leaks" is a web challenge focused on identifying and exploiting real-world web vulnerabilities. The target is a portal presenting supposedly confidential football data. Tools used: `nmap`, `curl`, `sqlmap`, and browser developer tools.

---

## Solution Walkthrough

### Step 1 — Initial Reconnaissance
Navigated to the Football Leaks application. The portal presented a restricted area and various documents — some legitimate, some distractions (including a RickRoll in ASCII format).

Inspected the HTML source and found **credentials exposed in a comment** — an emergency maintenance access that was never removed before publishing.

Port scan:
```bash
nmap -sC -sV -p- 172.30.1.37
```

Also ran a provided `programa.py` script locally — it contained encoded content and misleading messages acting as a distraction.

---

### Flag 1 — Client-Side Authentication Bypass

Analysed the login form with browser Developer Tools. The password field had a `required` attribute enforcing client-side validation.

**Bypass:** Removed the `required` attribute directly in the HTML via Developer Tools, then submitted the form without a password. The application accepted the request and granted access to the restricted area.

> **Security note:** Client-side validation is never sufficient. The server must always re-validate all inputs, especially authentication checks. Users control their browser and can modify HTML or send requests directly to the server.

**Flag 1:** `CTF{FL04T_L1K3_4_B4LL_S1NK_L1K3_4_L34K}`

---

### Flag 2 — IDOR (Insecure Direct Object Reference)

After gaining access, analysed the profile page. The user identifier was passed via a predictable URL parameter:
```
http://footballeaks.ctf/perfil.php?id=1
```

Manually changed `id=1` to other values to access profiles belonging to other users — no server-side authorisation check was in place.

Tested for SQL injection with sqlmap (no results), and checked HTTP headers with curl:
```bash
sqlmap -u "http://footballeaks.ctf/perfil.php?id=1" --dbs
curl -i "http://footballeaks.ctf/"
```

Accessing another user's profile revealed the second flag.

> **Security note:** IDOR allows users to access resources belonging to others by modifying a predictable identifier. The fix: always verify server-side that the authenticated user has permission to access the requested resource.

**Flag 2:** `CTF{1DOR_PRIV_3SC_RU1_P1NT0}`

---

### Flag 3 — SQL Injection (UNION-based)

Found a player search feature in the restricted area. Testing the search field revealed that user input was not sanitised before being used in a database query.

Constructed a manual `UNION SELECT` to query the `contratos_confidenciais` table:
```sql
' UNION SELECT 11,
CONCAT_WS(
' | ',
id, jogador_id, clube, salario_anual_eur, duracao_anos,
agente, notas_confidenciais, bonus_titulos, clausula_rescisao,
comissao_agente_pct, data_inicio, data_fim
),
3,4,5,6,7
FROM contratos_confidenciais
WHERE id=11 -- -
```

The query successfully extracted confidential contract data including the third flag.

> **Security note:** SQL injection allows manipulation of database queries and can lead to data exposure, modification, or deletion. The fix: use parameterised queries (prepared statements), validate all inputs, and apply the principle of least privilege to the database account.

**Flag 3:** `CTF{SQL_1NJ3CT10N_3XP0S3D_C0NTR4CT5}`

---

## Flags

| # | Vulnerability | Flag |
|---|--------------|------|
| 1 | Client-Side Auth Bypass | `CTF{FL04T_L1K3_4_B4LL_S1NK_L1K3_4_L34K}` |
| 2 | IDOR | `CTF{1DOR_PRIV_3SC_RU1_P1NT0}` |
| 3 | SQL Injection | `CTF{SQL_1NJ3CT10N_3XP0S3D_C0NTR4CT5}` |
