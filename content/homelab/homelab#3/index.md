---
title: "Homelab #3: Project STARK — Automated Media Server on Proxmox"
date: 2026-09-29
description: "The SFF server finally arrived. Building a fully automated media stack on Proxmox VE with Docker, LXC, Jellyfin, Radarr, Sonarr, and a 2TB media pool."
---

## The Server Has Arrived

The GMKtec SFF machine I'd been waiting for finally landed. The Raspberry Pi had been doing its best, but there's only so much you can ask of a single-board computer, so it was time to build something real.

I had a name ready before the hardware even shipped: **Project STARK**. I'm a Marvel fan, and the idea of building a self-hosted, automated, always-on infrastructure felt like it deserved a name that matched the ambition. I think Tony would approve.

The goal was a fully automated media factory. Something that could find, download, organise, and serve content with minimal manual intervention. Here's how it came together.

---

## Architecture

Being totally honest about this. Proxmox was the steepest learning curve of the entire project. Not because it's poorly documented, but because it introduces concepts (hypervisors, LXC containers, LVM volumes, bind mounts) that are genuinely new if you've only worked with VirtualBox or Docker before. I spent more time understanding *why* things worked than actually making them work.

Once it clicked, though, it clicked properly. Here's what the stack looks like:

| Layer | Technology |
|-------|-----------|
| Hypervisor | Proxmox VE (node: `stark`) |
| Virtualisation | LXC Container (Debian 12) |
| OS Storage | NVMe (system + container databases) |
| Media Storage | 2TB HDD - LVM volume, EXT4 |
| Orchestration | Docker + Docker Compose |
| Media Stack | Jellyfin, Radarr, Sonarr, Prowlarr, SABnzbd, Jellyseerr |

---

## Step 1 - Storage Provisioning

The 2TB drive came formatted for Windows. First task was wiping the partitions and integrating it properly into the Linux ecosystem.

I created a Logical Volume and formatted it in EXT4 for full POSIX permission compatibility:

```bash
# After creating the LVM volume group and logical volume
mkfs.ext4 /dev/mapper/mediapool-mediavolume
mount /dev/mapper/mediapool-mediavolume /mnt/media
```

Rather than virtualising the disk into a `.raw` or `.qcow2` file (which adds overhead), I set up a direct bind mount between the Proxmox host and the LXC container:

```bash
pct set 101 -mp0 /mnt/media,mp=/media
```

This gives the container raw access to the storage with no virtualisation layer in the way, which is important when you're moving large media files constantly.

---

## Step 2 - LXC Configuration for Docker

Running Docker inside an unprivileged LXC container requires one specific flag that isn't obvious until you hit the wall:

```bash
pct set 101 -features nesting=1
```

Without nesting enabled, the Docker daemon can't create its own containers inside the LXC base. Once that was set, the directory structure came together:

- `/opt/stark-media` - on the NVMe, for config files and databases
- `/media` - bind mounted from the 2TB pool

---

## Step 3 - Docker Compose Deploy

The entire stack is declared in a single `docker-compose.yml`. Key details:

- All services use `TZ=Europe/Lisbon` for correct timestamps
- Config volumes mapped to `/opt/stark-media/config/<service>`
- Media volumes mapped to `/media`
- All `*arr` services communicate over Docker's internal network, resolving each other by container name

```bash
docker compose up -d
```

One command, six services, everything was working!

---

## Step 4 - Automation Pipeline

The media factory runs like an assembly line:

1. **Jellyseerr** - user-facing request interface. You search for a film or series and request it.
2. **Radarr / Sonarr** - pick up the request and search for a matching release.
3. **Prowlarr** - the central indexer manager, authenticated to both Radarr and Sonarr via API keys with Full Sync.
4. **SABnzbd** - handles the actual download.
5. **Jellyfin** - picks up the finished file, scrapes metadata, and serves it.

All inter-service communication happens over the Docker internal network. Nothing is exposed unnecessarily.

**Jellyfin extras:**
- **Trakt.tv plugin** - bidirectional watch history scrobbling
- **Comic Vine plugin** - automatic metadata for `.cbz` and `.cbr` comic files

Yes, there's a comics library (This is a future project). STARK.link has layers.

---

## Step 5 - Cross-VLAN SMB Share

To migrate my existing media files from a Windows 11 workstation, I needed a network share accessible across VLANs. The server sits on a dedicated server VLAN, the workstation on the personal VLAN.

I set up Samba directly in the LXC, with a dedicated system user instead of anonymous access (which modern Windows blocks by default):

```bash
useradd stark-share
smbpasswd -a stark-share
```

Obs: Not my real user or password!

Key `smb.conf` settings:
```ini
valid users = stark-share
force user = root
```

The `force user = root` flag ensures files written by the Windows client don't create permission conflicts with the Docker processes (Jellyfin and Radarr need to read and write the same files).

---

## What I Learned

Proxmox has a real learning curve, and I'm not going to pretend otherwise. Coming from VirtualBox, the jump to a proper hypervisor with LXC containers, LVM storage, and network isolation felt significant, but it's the right tool for this kind of infrastructure and I'm glad I pushed through.

Project STARK is now online, but not complete. Jellyfin is running, the automation pipeline works end to end, the comics library exists, and the 2TB pool is filling up nicely.

Next steps: integrating Wazuh monitoring into the STARK environment so the SIEM from the previous post can watch this infrastructure. JARVIS would want visibility.