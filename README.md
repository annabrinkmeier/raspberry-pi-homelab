# Raspberry Pi Home Lab

> A self-built home lab using three Raspberry Pi devices to develop practical IT infrastructure, networking, and cybersecurity skills.

![Homelab Banner](https://img.shields.io/badge/Status-In%20Progress-yellow?style=for-the-badge)
![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-4%20Devices-C51A4A?style=for-the-badge&logo=raspberry-pi)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Lab Architecture](#lab-architecture)
- [Pi 1 — RetroPie Gaming Server](#pi-1--retropie-gaming-server)
- [Pi 2 — Pi-hole + WireGuard VPN](#pi-2--pi-hole--wireguard-vpn)
- [Pi 3 — Pi Movie Vault](#pi-3--pi-movie-vault)
- [Pi 4 — Security Operations](#pi-4--security-operations)
- [Progress Tracker](#progress-tracker)
- [Future Plans](#future-plans)

---

## Overview

This project documents my hands-on home lab built with four Raspberry Pi single-board computers. Each Pi is configured to serve a distinct purpose, collectively covering a wide range of IT and cybersecurity disciplines including network administration, DNS management, VPN configuration, containerization, intrusion prevention, and SIEM deployment.

**Why I built this:** 
Real skills come from breaking things, fixing them, and understanding why they broke. This lab is my sandbox for doing exactly that.

---
```
Home Network (192.168.x.x/24)
│
├── Router / Gateway
│   └── DHCP → Pi-hole set as network-wide DNS
│
├── Pi 1 — RetroPie Gaming
│   └── Services: EmulationStation, RetroArch
│
├── Pi 2 — DNS / VPN / Security
│   ├── Pi-hole (DNS sinkhole, port 53)
│   ├── WireGuard VPN (UDP [custom port])
│   └── Fail2ban (SSH intrusion prevention)
│
├── Pi 3 — Media Server
│   ├── Jellyfin (port 8096)
│   └── Nextcloud (planned)
│
└── Pi 4 — Security Operations (In Progress)
    └── Wazuh SIEM
```

---

## Pi 1 — RetroPie Gaming Server

### Purpose
Configure a dedicated retro gaming console using RetroPie — establishing fundamentals of OS installation, SSH access, and system configuration.

### What I Did
- Installed RetroPie image via Raspberry Pi Imager
- Enabled SSH and configured WiFi headlessly
- Changed default credentials immediately
- Enabled key-based SSH authentication, disabled password login
- Configured UFW firewall rules
- Reflashed when package conflicts made the system unrecoverable

### Skills Practiced
`Linux CLI` · `SSH` · `Headless Setup` · `UFW Firewall` · `SSH Hardening`

### What I Learned
- How to flash and configure an OS image headlessly
- How to generate SSH keypairs and implement key-based authentication
- How Debian package repositories work and how to fix broken ones
- How to configure UFW firewall rules
- Sometimes a clean reinstall is the right call — knowing when to cut losses is a real sysadmin skill

**Known Issue:**
RetroPie 4.8 is built on Debian Buster which has package conflicts preventing full system upgrades. Unattended-upgrades could not be installed due to held packages (libgcc-8-dev conflict). Manual updates via `sudo apt update && sudo apt upgrade` work for individual packages.

---

## Pi 2 — Pi-hole + WireGuard VPN

### Purpose
Transform a Raspberry Pi into a network-wide ad blocker, personal VPN server, and intrusion prevention system.

### Pi-hole Configuration
- Installed Pi-hole v6 via official installer
- Set as network-wide DNS via router (Plume) — all 17 devices protected automatically
- Blocking **543,587 domains** (StevenBlack + HaGeZi Pro blocklists)
- Currently blocking **~25% of all network traffic**
- Whitelisted Apple iCloud Private Relay to maintain iPhone functionality
- Identified and blocked Roku TV surveillance (Alphonso.tv), Amazon Alexa telemetry, Sift Science behavioral tracking, and Datadog browser monitoring
- Configured static IP via NetworkManager (nmcli)

### WireGuard VPN Configuration
- Generated public/private keypairs for server and clients using Curve25519
- Configured `wg0.conf` interface and peer definitions from scratch
- Set up IP forwarding (`net.ipv4.ip_forward=1`) — made permanent via `/etc/sysctl.d/`
- Configured UFW and iptables firewall rules
- Set up port forwarding on router (UDP 51820)
- Connected iOS client via QR code
- Configured on-demand activation — auto-connects on cellular, auto-disconnects on home WiFi

### Fail2ban Configuration
- Installed and configured Fail2ban for SSH intrusion prevention
- Ban policy: configured with custom ban policies
- Actively monitoring SSH login attempts across all Pis

### Skills Practiced
`DNS Administration` · `Network-wide Filtering` · `VPN Configuration` · `Public Key Cryptography` · `Firewall Rules (UFW/iptables)` · `IP Forwarding` · `Port Forwarding` · `Intrusion Prevention`

### What I Learned
- How DNS sinkholing works at a network level
- DNS single points of failure — configured 1.1.1.1 as fallback DNS
- Pi-hole v6 completely changed the web interface — had to troubleshoot using curl and systemctl
- IP forwarding resets to 0 on reboot unless saved to `/etc/sysctl.d/`
- A single character typo (`erth0` instead of `eth0`) broke VPN routing for months — fixed by carefully auditing config files character by character
- Difference between IDS (detection) and IPS (prevention) — Fail2ban is active prevention
- Pi-hole cannot block YouTube ads because Google serves ads from the same domains as video content
- Smart devices (Alexa, Roku) constantly send telemetry — Pi-hole can block surveillance while maintaining functionality

---

## Pi 3 — Pi Movie Vault

### Purpose
Self-hosted media server using Docker on a Raspberry Pi 5.

### Current Status: Jellyfin Complete ✅ | Nextcloud Planned ⬜

### Completed
**Jellyfin Media Server** ✅
- Installed via Docker on Raspberry Pi OS Lite (64-bit)
- Connected 3.6TB external SSD for media storage
- Configured auto-mount via fstab (exFAT format)
- Accessible on local network via port 8096
- Streaming to Apple TV via Swiftfin app
- Currently converting ISO disc images to MKV using MakeMKV and FileBot for proper episode organization
- Connected via ethernet for stable streaming

### Planned
**Nextcloud (Self-hosted Cloud)**
- Deploy via Docker Compose with MariaDB backend
- Configure HTTPS using self-signed certificate
- Set up Nginx reverse proxy
- Enable automatic backups via cron job

### What I Learned
- Docker installation and container management
- Volume mounts and persistent storage in Docker
- exFAT filesystem mounting and fstab configuration
- Hotel WiFi uses client isolation — devices on the same network can't communicate (discovered during remote setup attempt)
- Ethernet vs WiFi for media streaming reliability

---

## Pi 4 — Security Operations Center (In Progress)

### Purpose
Dedicated Wazuh SIEM server to collect, correlate, and analyze security logs from all other Pis — building a real Security Operations Center.

### Current Status: ⬜ Not Started

### Planned Setup
- Deploy Wazuh SIEM server
- Install Wazuh agents on Pi 1, 2, and 3
- Configure log collection and alerting
- Build security dashboards

---

## Progress Tracker

| Milestone | Status | Date Completed |
|-----------|--------|----------------|
| Pi 1: OS installed & SSH configured | ✅ | March 2026 |
| Pi 1: RetroPie running | ✅ | March 2026 |
| Pi 1: WiFi connected | ✅ | March 2026 |
| Pi 1: Default password changed | ✅ | March 2026 |
| Pi 1: SSH enabled & connected | ✅ | March 2026 |
| Pi 1: SSH hardening applied | ✅ | March 2026 |
| Pi 1: Reflashed with fresh RetroPie image | ✅ | March 2026 |
| Pi 1: UFW firewall configured | ✅ | March 2026 |
| Pi 1: System updates run | ⬜ | |
| Pi 2: OS installed & SSH configured | ✅ | March 2026 |
| Pi 2: Pi-hole installed & network DNS set | ✅ | March 2026 |
| Pi 2: Blocklists configured | ✅ | March 2026 |
| Pi 2: UFW firewall configured | ✅ | March 2026 |
| Pi 2: Web dashboard accessible | ✅ | March 2026 |
| Pi 2: WireGuard server configured | ✅ | March 2026 |
| Pi 2: WireGuard client configured | ✅ | March 2026 |
| Pi 2: Port forwarding configured | ✅ | April 2026 |
| Pi 2: Mobile VPN client connected | ✅ | April 2026 |
| Pi 2: VPN routing fixed (eth0 typo) | ✅ | May 2026 |
| Pi 2: Pi-hole static IP configured | ✅ | May 2026 |
| Pi 2: Roku TV added to Pi-hole | ✅ | May 2026 |
| Pi 2: iPhone added to Pi-hole | ✅ | May 2026 |
| Pi 2: IP forwarding made permanent | ✅ | May 2026 |
| Pi 2: WireGuard on-demand cellular configured | ✅ | May 2026 |
| Pi 2: Blocklists updated to 543k domains | ✅ | May 2026 |
| Pi 2: Network-wide DNS via Plume configured | ✅ | May 2026 |
| Pi 2: 17 devices protected by Pi-hole | ✅ | May 2026 |
| Pi 2: Fail2ban installed and configured | ✅ | May 2026 |
| Pi 3: OS installed & SSH configured | ✅ | April 2026 |
| Pi 3: Docker installed | ✅ | April 2026 |
| Pi 3: Jellyfin installed via Docker | ✅ | April 2026 |
| Pi 3: Media SSD mounted (3.6TB) | ✅ | April 2026 |
| Pi 3: Jellyfin accessible on Apple TV | ✅ | April 2026 |
| Pi 3: SSD auto-mount configured | ✅ | April 2026 |
| Pi 3: Nextcloud running with HTTPS | ⬜ | |
| Pi 3: Automatic backups configured | ⬜ | |
| Pi 4: Wazuh SIEM deployed | ⬜ | |
| Pi 4: Wazuh agents on all Pis | ⬜ | |
| All: Fail2ban on all Pis | 🔄 | |
| Documentation complete | 🔄 | |

✅ Complete · 🔄 In Progress · ⬜ Not Started

---
## Future Plans

### Immediate Next Steps
- [ ] Complete Wazuh SIEM deployment on Pi 4
- [ ] Install Fail2ban on Pi 3 and Pi 4
- [ ] Deploy Nextcloud on Pi 3
- [ ] Set up Grafana + Prometheus monitoring dashboards
- [ ] Add honeypot to observe real attack patterns
- [ ] Configure centralized logging across all Pis
- [ ] Experiment with Kubernetes (k3s)
- [ ] Add DMZ network segment

### Security+ Lab Roadmap
Structured additions aligned with Security+ exam domains:

**Current ✅**
- Pi-hole DNS sinkhole (Domain 2 — Threat Intelligence)
- WireGuard VPN (Domain 3 — Network Security)
- Jellyfin in Docker (Domain 3 — Containerization)
- UFW Firewall (Domain 3 — Firewall Rules)
- SSH Hardening + Key-based Auth (Domain 1 — Authentication)
- Fail2ban IPS (Domain 2 — Intrusion Prevention)

**After Lesson 3 ⬜**
- Wazuh SIEM on Pi 4 (Domain 4 — Log Analysis)
- VLANs via Plume (Domain 3 — Network Segmentation)

**After Lesson 4 ⬜**
- Snort or Suricata IDS (Domain 4 — Intrusion Detection)
- Integrate IDS alerts into Wazuh SIEM

**After Lesson 5 ⬜**
- Formal security policy document
- Incident response procedure
- Complete network diagram with security controls labeled
---

## About

Built and maintained by Anna Brinkmeier as part of a self-directed IT and cybersecurity learning journey.

🔗 [LinkedIn](https://www.linkedin.com/in/anna-brinkmeier-246b34345)
🐙 [GitHub](https://github.com/annabrinkmeier)

---

*"The best way to learn infrastructure is to build it, break it, and document why it broke."*
