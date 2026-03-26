# Raspberry Pi Home Lab

> A self-built home lab using three Raspberry Pi devices to develop practical IT infrastructure, networking, and cybersecurity skills.

![Homelab Banner](https://img.shields.io/badge/Status-In%20Progress-yellow?style=for-the-badge)
![Raspberry Pi](https://img.shields.io/badge/Raspberry%20Pi-3%20Devices-C51A4A?style=for-the-badge&logo=raspberry-pi)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

---

## 📋 Table of Contents

- [Overview](#overview)
- [Lab Architecture](#lab-architecture)
- [Pi 1 — RetroPie Gaming Server](#pi-1--retropie-gaming-server)
- [Pi 2 — Pi-hole + WireGuard VPN](#pi-2--pi-hole--wireguard-vpn)
- [Pi 3 — Jellyfin + Nextcloud](#pi-3--jellyfin--nextcloud)
- [Skills Demonstrated](#skills-demonstrated)
- [Security Practices](#security-practices)
- [What I Learned](#what-i-learned)
- [Progress Tracker](#progress-tracker)
- [Setup Guides](#setup-guides)
- [Future Plans](#future-plans)

---

## Overview

This project documents my hands-on home lab built with three Raspberry Pi single-board computers. Each Pi is configured to serve a distinct purpose, collectively covering a wide range of IT and cybersecurity disciplines. From network administration and DNS management to VPN configuration, containerization, and self-hosted cloud services.

**Why I built this:** Real skills come from breaking things, fixing them, and understanding why they broke. This lab is my sandbox for doing exactly that.

---

## Lab Architecture

```
Home Network (192.168.1.0/24)
│
├── Router / Gateway (192.168.1.1)
│   └── DHCP → Assigns static leases to each Pi
│
├── Pi 1 — RetroPie        [192.168.1.101]
│   └── Services: EmulationStation, RetroArch
│
├── Pi 2 — DNS / VPN       [192.168.1.102]
│   ├── Pi-hole (DNS sinkhole, port 53)
│   └── WireGuard VPN (UDP 51820)
│
└── Pi 3 — Media / Cloud   [192.168.1.103]
    ├── Jellyfin (port 8096)
    └── Nextcloud (port 443 / Docker)
```

**All Pis run Raspberry Pi OS (64-bit, Lite or Full depending on use case)**

---

## Pi 1 — RetroPie Gaming Server

### Purpose
Configure a dedicated retro gaming console using RetroPie — a beginner-friendly Linux project that establishes fundamentals of OS installation, SSH access, file transfer, and system configuration.

### Key Configurations
- Installed RetroPie image via Raspberry Pi Imager
- Enabled SSH and configured Wi-Fi headlessly via `wpa_supplicant.conf`
- Transferred ROM files via SCP / SFTP (FileZilla)
- Configured controller mappings and display resolution via `raspi-config`
- Set up static IP via DHCP reservation on router

### Skills Practiced
`Linux CLI` · `SSH` · `SCP/SFTP` · `Headless Setup` · `File Permissions` · `Network Configuration`

### Cybersecurity Concepts Learned
- Changed default `pi` password and disabled default user
- Enabled key-based SSH authentication, disabled password login
- Understood attack surface reduction (disabled unused services)
- How to flash an OS image using Raspberry Pi Imager
- How to SSH into a Linux device from a Mac
- How to generate SSH keypairs and use key-based authentication
- How to disable password authentication for SSH hardening
- How to configure UFW firewall rules
- How Debian package repositories work and how to fix broken ones
- Troubleshooting is a core IT skill — sometimes a fresh install 
  is the right decision
- Learned about DNS single points of failure firsthand
— configured a fallback DNS server (1.1.1.1) to prevent 
internet outages if Pi-hole becomes unavailable.

**Known Issue:** 
RetroPie 4.8 is built on Debian Buster which has 
package conflicts preventing full system upgrades. Unattended-upgrades 
could not be installed due to held packages (libgcc-8-dev conflict). 
Manual updates via `sudo apt update && sudo apt upgrade` work for 
individual packages. A future improvement would be migrating to a 
Bookworm-based image when RetroPie releases one.
---

## Pi 2 — Pi-hole + WireGuard VPN

### Purpose
Transform a Raspberry Pi into a network-wide ad blocker and personal VPN server — covering DNS, networking, firewall rules, and encrypted tunneling.

### Pi-hole Configuration
- Installed Pi-hole via official installer
- Set as primary DNS server on router (network-wide blocking)
- Added community blocklists (StevenBlack, OISD)
- Configured custom DNS records for local hostnames
- Reviewed query logs to understand DNS traffic patterns

### WireGuard VPN Configuration
- Generated public/private keypairs for server and clients
- Configured `wg0.conf` interface and peer definitions
- Set up IP forwarding (`net.ipv4.ip_forward=1`)
- Configured UFW firewall rules to allow VPN traffic
- Enabled split tunneling vs. full tunnel configurations
- Set up port forwarding on router (UDP 51820)
- Connected iOS and Android clients via QR code config

### Skills Practiced
`DNS Administration` · `Network-wide Filtering` · `VPN Configuration` · `Public Key Cryptography` · `Firewall Rules (UFW/iptables)` · `IP Forwarding` · `Port Forwarding`

### Cybersecurity Concepts Learned
- How DNS-based ad/malware blocking works (DNS sinkholing)
- Asymmetric key cryptography in WireGuard (Curve25519)
- Principle of least privilege in firewall rules
- Traffic analysis via Pi-hole query logs
- VPN tunneling and encrypted data in transit
### What I Learned on Pi 2

**Pi-hole & DNS:**
- How DNS works at a network level and how it can be used 
  for security (blocking malicious domains before they load)
- Configured a DNS sinkhole blocking 77,526 ad and malware domains
- Learned about DNS single points of failure — added 1.1.1.1 
  as a fallback DNS to prevent internet outages
- Discovered Pi-hole v6 changed the web interface significantly
  — had to troubleshoot using curl and systemctl to diagnose

**WireGuard VPN:**
- How VPN tunneling works at a cryptographic level using 
  Curve25519 asymmetric key pairs
- Generated server and client keypairs manually
- Configured wg0.conf interface and peer definitions from scratch
- Understood the difference between local and external VPN testing
- Learned that port forwarding is required for external VPN access

**Security & Networking:**
- Configured UFW firewall rules for SSH, WireGuard, and web ports
- Enabled IP forwarding for VPN traffic routing
- Learned how self-signed SSL certificates work and why browsers 
  block them
- Troubleshot network connectivity using ping, curl, ss, and systemctl
- Understood NAT (Network Address Translation) and port forwarding

**Real World Lesson:**
- Troubleshooting is 90% of IT work — this Pi required diagnosing 
  multiple issues including missing packages, firewall blocks, 
  and a major version change in Pi-hole
---
## Pi 3 — Pi Movie Vault (Jellyfin + Nextcloud)
### Purpose
Host a personal media server and self-hosted cloud storage solution 
using Docker — covering containerization, reverse proxying, SSL 
certificates, and storage management.

### Current Status: In Progress 🔄

### What I've Done So Far
- Flashed Raspberry Pi OS Lite (64-bit) using Raspberry Pi Imager
- Pre-configured hostname (pimovievault), username, SSH keys, 
  and WiFi credentials during flash using Imager customisation settings
- Attempted remote setup from hotel during work trip
- Discovered hotel WiFi uses **client isolation** — a security feature 
  that prevents devices on the same network from communicating.
  Confirmed by running a full network sweep (arp -a and ping sweep)
  with no Pi hostname appearing in results.
- SD card is flashed and ready for home network deployment

### Planned Setup
**Jellyfin Media Server**
- Install via Docker Compose
- Configure media library with correct volume mounts
- Set up hardware transcoding
- Accessible on local network at `http://192.168.40.xxx:8096`

**Nextcloud (Self-hosted Cloud)**
- Deploy via Docker Compose with MariaDB backend
- Configure HTTPS using self-signed certificate
- Set up Nginx reverse proxy for clean domain routing
- Configure file storage and user accounts
- Enable automatic backups via cron job

### Planned Skills to Practice
`Docker & Docker Compose` · `Reverse Proxy (Nginx)` · `SSL/TLS Certificates` · `Database Administration (MariaDB)` · `Cron Jobs` · `Self-hosting` · `Storage Management`

---

## Progress Tracker

| Milestone | Status | Date Completed |
|-----------|--------|----------------|
| Pi 1: OS installed & SSH configured | ✅ | March 2026 |
| Pi 1: RetroPie running | ✅ | March 2026 |
| Pi 1: WiFi connected | ✅ | March 2026 |
| Pi 1: Default password changed | ✅ | March 2026 |
| Pi 1: SSH enabled & connected | ✅ | March 2026 |
| Pi 1: System updates run |  |  |
| Pi 1: SSH hardening applied | ✅ | March 2026 |
| Pi 1: Reflashed with fresh RetroPie image | ✅ | March 2026 |
| Pi 1: UFW firewall configured | ✅ | March 2026 |
| Pi 1: SSH  enabled |  | |
| Pi 2: OS installed & SSH configured | ✅ | March 2026 |
| Pi 2: Pi-hole installed & network DNS set | ✅ | March 2026 |
| Pi 2: Blocklists configured | ✅ | March 2026 |
| Pi 2: Set as DNS on Mac and iPhone | ✅ | March 2026 |
| Pi 2: UFW firewall configured | ✅ | March 2026 |
| Pi 2: Web dashboard accessible | ✅ | March 2026 |
| Pi 2: WireGuard server configured | ✅ | March 2026 |
| Pi 2: WireGuard client configured | ✅ | March 2026 |
| Pi 2: Mobile VPN client connected | ⬜ | |
| Pi 2: Port forwarding configured (i3 call) | ⬜ | |
| Pi 3: Docker installed | ⬜ | |
| Pi 3: Jellyfin running in Docker | ⬜ | |
| Pi 3: Nextcloud running with HTTPS | ⬜ | |
| Pi 3: Automatic backups configured | ⬜ | |
| All: UFW firewall configured | ⬜ | |
| All: fail2ban installed | ⬜ | |
| Documentation complete | 🔄 | |

✅ Complete · 🔄 In Progress · ⬜ Not Started

---

## Setup Guides

Detailed step-by-step guides for each Pi are available in the `/docs` folder:

- [`docs/pi1-retropie-setup.md`](docs/pi1-retropie-setup.md)
- [`docs/pi2-pihole-setup.md`](docs/pi2-pihole-setup.md)
- [`docs/pi2-wireguard-setup.md`](docs/pi2-wireguard-setup.md)
- [`docs/pi3-jellyfin-docker-setup.md`](docs/pi3-jellyfin-docker-setup.md)
- [`docs/pi3-nextcloud-docker-setup.md`](docs/pi3-nextcloud-docker-setup.md)
- [`docs/security-hardening-checklist.md`](docs/security-hardening-checklist.md)

---

## Future Plans

- [ ] Set up a SIEM (Security Information & Event Management) using Wazuh or Graylog
- [ ] Add a fourth Pi as a honeypot to observe attack patterns
- [ ] Configure centralized logging across all Pis
- [ ] Set up Grafana + Prometheus for system monitoring dashboards
- [ ] Experiment with Kubernetes (k3s) for container orchestration
- [ ] Add a DMZ network segment for internet-exposed services

---

## About

Built and maintained by Anna Brinkmeier as part of a self-directed IT and cybersecurity learning journey.

📧 annaebrinkmeier@gmail.com
🔗 [LinkedIn](https://www.linkedin.com/in/anna-brinkmeier-246b34345)
🐙 [GitHub](https://github.com/annabrinkmeier)

---

*"The best way to learn infrastructure is to build it, break it, and document why it broke."*
