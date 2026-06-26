---
title: "Homelab #2: Wazuh SIEM/XDR - Attack Simulation & Active Response"
date: 2026-06-26
description: "Building a more advanced Wazuh lab on VirtualBox with Sysmon integration, real attack simulations, and automated Active Response until I hit the wall of my current skill level."
---

## Why This Lab?

After setting up the basic Wazuh stack with Docker, I wanted to push further. My course had given me a solid overview of SIEM concepts, but I wanted to see how far I could take it on my own and building something more complex, simulating real attacks, and watching the detections fire in real time.

I got pretty far. I set up a proper isolated lab, integrated Sysmon for deeper telemetry, ran two controlled attack scenarios from the MITRE ATT&CK framework, and even got Active Response working automatically blocking an attacker at the firewall level within milliseconds of detection.

Then I hit a wall. There were things I wanted to do next that I simply wasn't good enough yet to implement cleanly. So this post documents how far I got, what I learned, and what's left for when I come back to it.

---

## Architecture

The lab runs entirely in VirtualBox with an isolated internal network to make sure no test traffic could reach my actual home network.

```
+-------------------------------------------------------+
|                    VIRTUALBOX                         |
|                                                       |
|   +-----------------------+   +-------------------+   |
|   | VM 1: Debian Server   |   | VM 2: Windows 10  |   |
|   | (Wazuh Manager)       |   | (Victim Endpoint) |   |
|   | IP: 192.168.50.10     |   | IP: 192.168.50.20 |   |
|   +-----------+-----------+   +---------+---------+   |
|               |                         |             |
+---------------|-------------------------|-------------+
                |     Internal Network    |
                +------------X------------+
```


**VM 1 - Wazuh Manager & Indexer**
Debian Server 13.5.0, 4 vCPUs, 8GB RAM, 50GB disk. Handles log ingestion, parsing, rule correlation, and index storage.

**VM 2 - Victim Endpoint**
Windows 10 Enterprise, 2 vCPUs, 4GB RAM. Simulates a standard corporate workstation with the Wazuh Agent installed.

---

## Going Beyond Basic Logs - Sysmon Integration

Windows Security Event Logs have critical blind spots. They don't natively correlate process lineage or network connections in a useful way for threat detection. To fix this, I integrated **Microsoft Sysmon** on the Windows VM.

Install Sysmon with a modular config focused on attacker behaviours:
```dos
Sysmon64.exe -i sysmonconfig-modular.xml
```

This adds three key event types:

- **Event ID 1** - Process Creation (full command line, user, MD5/SHA256 hash of the binary)
- **Event ID 3** - Network Connections (which local process opened a socket to which external IP)
- **Event ID 11** - File Creation (detecting payload drops in temp folders)

To get these into Wazuh, I edited the agent config on the Windows VM (`C:\Program Files (x86)\ossec-agent\ossec.conf`) and added:

```xml
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

After restarting the agent service, structured XML events from Sysmon started flowing into the SIEM pipeline continuously.

---

## Attack Simulation

With telemetry in place, I ran two controlled attacks from a third machine to test the detection capability.

### Scenario 1 - RDP Brute Force (MITRE T1110.002)

**The attack:** An automated script generating 50 failed login attempts in under 20 seconds against the default RDP port (3389), using a dictionary of common admin usernames.

**What Wazuh saw:** The endpoint generated multiple Windows Event ID 4625 (failed logon) events. The base rule detected each failure, but the high-priority correlation rule (level 10+) fired when the frequency threshold within the timeframe was exceeded.

**Raw alert:**
```json
{
  "win": {
    "system": {
      "providerName": "Microsoft-Windows-Security-Auditing",
      "eventID": "4625"
    },
    "eventdata": {
      "logonType": "3",
      "targetUserName": "Administrator",
      "ipAddress": "192.168.50.99"
    }
  },
  "rule": {
    "id": "60122",
    "level": 10,
    "description": "Windows: Multiple logon failures - Possible Brute Force attack."
  }
}
```

From the dashboard I could immediately pull the attacker's IP (`192.168.50.99`), the authentication type (LogonType 3 - network/remote), and the target account (`Administrator`). No session was established.

---

### Scenario 2 - Post-Compromise Reconnaissance (MITRE T1033 / T1087)

**The attack:** Simulating an attacker who already has console access and runs quick enumeration commands: `whoami`, `net user`, and `ipconfig /all`.

**What Wazuh saw:** A traditional antivirus would ignore these entirely as they are legitimate native Windows tools. But Sysmon caught the full execution context.

**Process tree from Sysmon Event ID 1:**
- **Process created:** `whoami.exe`
- **Command line:** `C:\Windows\system32\whoami.exe`
- **Parent process:** `powershell.exe` with execution policy bypass arguments, indicating a Reverse Shell

The binary hash was automatically validated against the system integrity baseline, confirming the real `whoami.exe` was not tampered with but its execution context was clearly malicious.

Seeing this fire in real time was the highlight of the lab for me. Watching commands I ran myself get flagged and contextualised by the SIEM within seconds made everything I had read about behavioural detection actually click.

---

## Active Response - Automated Blocking

The most satisfying part of the lab. I configured Wazuh's Active Response to automatically block the attacking IP at the Windows Firewall level the moment the brute force rule fired, so no human intervention required.

**Configuration in the Wazuh Manager's `ossec.conf`:**
```xml
<!-- Block command definition -->
<command>
  <name>netsh-block-ip</name>
  <executable>netsh-w firewall-drop.cmd</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>

<!-- Trigger on brute force rule -->
<active-response>
  <command>netsh-block-ip</command>
  <location>local</location>
  <rules_id>60122</rules_id>
  <timeout>1800</timeout> <!-- 30 minute block -->
</active-response>
```

**What happened during the test:** The moment the 10th failed login hit the server, rule 60122 fired. The Manager sent an encrypted instruction back to the Windows agent, which executed:

```dos
netsh advfirewall firewall add rule name="Wazuh Block IP" dir=in action=block remoteip=192.168.50.99
```

The attacker's connection dropped instantly. After 30 minutes, the agent removed the rule automatically and restored the original network state.

The response time was measured in milliseconds compared to the minutes it would take a human analyst to read the alert and act manually.

---

## Where I Hit the Wall

I wanted to go further - integrating external Threat Intelligence APIs like VirusTotal to automatically enrich file creation alerts with global reputation data, and writing custom YARA rules to trigger on suspicious downloads.

I started down that path and quickly realised I didn't have enough knowledge yet to do it cleanly. Rather than cobble something together I didn't fully understand, I stopped here. This is the honest version of the lab: it goes as far as my current skill level allowed, and that's fine.

When I come back to this, once the SFF server is running and I have a proper Proxmox environment, I'll pick up from here.

---

## What I Learned

- **Context beats signatures.** Traditional security tools focused on signatures fail against behavioural attacks. Correlating Sysmon process metadata with Wazuh's rule engine is a far more robust defence against modern techniques.
- **Automation matters.** Active Response reduces mean time to remediation from minutes to milliseconds. That difference is significant in a real incident.
- **Knowing your limits is a skill.** Stopping before I broke something I didn't understand was the right call. I learned more from the parts that worked cleanly than I would have from forcing the parts that didn't.