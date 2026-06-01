---
title: "Homelab #1: Raspberry Pi 4 — Pi-hole, Tailscale & Jellyfin"
date: 2026-06-01
description: "How I turned a Raspberry Pi 4 into a home DNS filter, VPN node, and media server."
---
<!--more-->

## Overview

Before the SFF server arrives, my Raspberry Pi 4 is doing the heavy lifting.
It currently runs three services: Pi-hole for DNS-level ad and tracker blocking,
Tailscale for secure remote access, and Jellyfin as a self-hosted media server.

This post documents the setup and what I learned along the way.

---

## Hardware

- Raspberry Pi 4 (4GB RAM)
- MicroSD card (32GB)
- Raspberry Pi OS Lite (64-bit, headless)

---

## 1. Pi-hole — Network-wide DNS Filtering

Pi-hole acts as a local DNS resolver for the entire network. Any device that
uses the Pi as its DNS server gets ad and tracker domains blocked before the
request even leaves the network.

**Installation:**
```bash
curl -sSL https://install.pi-hole.net | bash
```

After setup, I pointed my router's DNS to the Pi's local IP so all devices
benefit automatically.

---

## 2. Tailscale — Zero-config VPN

Tailscale creates a private mesh network using WireGuard under the hood. With
it installed on the Pi and my other devices, I can reach home services securely
from anywhere without opening any ports on my router.

No exposed ports means no attack surface for port scanners. All traffic is
encrypted end-to-end via WireGuard. It's a good example of a zero-trust
approach applied at home scale.

**Installation:**
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

---

## 3. Jellyfin — Self-hosted Media Server

Jellyfin is an open-source alternative to Plex. It serves media files over the
local network (and remotely via Tailscale) without phoning home to any external
service.

**Installation:**
```bash
curl https://repo.jellyfin.org/install-jellyfin.py | sudo python3
```

Access via browser at my Pi's local or Tailscale IP on port 8096.

---

## What I Learned

- Running services on a single device requires thinking about resource contention
— Pi-hole is very lightweight, but Jellyfin transcoding can spike CPU on a Pi 4.
- Combining Pi-hole with Tailscale means I can use my home DNS filter even when
away from home by routing DNS through the Pi via the VPN.

---

## What's Next

An SFF machine is on the way. Once it arrives, I'll migrate Jellyfin, create a redundancy and start adding a proper SIEM (likely Wazuh) to monitor the
whole setup.