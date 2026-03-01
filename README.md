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

This project documents my hands-on home lab built with three Raspberry Pi single-board computers. Each Pi is configured to serve a distinct purpose, collectively covering a wide range of IT and cybersecurity disciplines — from network administration and DNS management to VPN configuration, containerization, and self-hosted cloud services.

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

---

## Pi 3 — Jellyfin + Nextcloud

### Purpose
Host a personal media server and self-hosted cloud storage solution using Docker — covering containerization, reverse proxying, SSL certificates, and storage management.

### Jellyfin Media Server
- Installed via Docker Compose
- Configured media library with correct volume mounts
- Set up hardware transcoding (if supported)
- Accessible on local network at `http://192.168.1.103:8096`

### Nextcloud (Self-hosted Cloud)
- Deployed via Docker Compose with MariaDB backend
- Configured HTTPS using Let's Encrypt (Certbot) or self-signed cert
- Set up Nginx reverse proxy for clean domain routing
- Configured file storage quotas and user accounts
- Enabled automatic backups via cron job

### Skills Practiced
`Docker & Docker Compose` · `Reverse Proxy (Nginx)` · `SSL/TLS Certificates` · `Database Administration (MariaDB)` · `Cron Jobs` · `Self-hosting` · `Storage Management`

### Cybersecurity Concepts Learned
- TLS encryption and certificate management
- Container isolation and security
- Authentication hardening (2FA in Nextcloud)
- Data sovereignty vs. cloud provider trust
- Backup strategies and data integrity

---

## Skills Demonstrated

| Domain | Skills |
|--------|--------|
| **Linux** | CLI navigation, file permissions, systemd services, cron, shell scripting |
| **Networking** | DNS, DHCP, static IPs, port forwarding, subnetting, firewall rules |
| **Security** | SSH hardening, VPN, firewall configuration, principle of least privilege |
| **Protocols** | DNS, HTTP/HTTPS, WireGuard, SSH, SFTP |
| **Containerization** | Docker, Docker Compose, volume mounts, networking |
| **Self-hosting** | Reverse proxies, SSL certs, domain routing, service management |
| **Documentation** | Architecture diagrams, setup guides, README writing |

---

## Security Practices

Applied consistently across all Pis:

- [ ] Changed default credentials on first boot
- [ ] Disabled default `pi` user account
- [ ] Enabled SSH key-based authentication
- [ ] Disabled SSH password authentication
- [ ] Configured UFW firewall (deny by default, allow specific ports)
- [ ] Enabled automatic security updates (`unattended-upgrades`)
- [ ] Assigned static IPs / DHCP reservations
- [ ] Enabled fail2ban on SSH-exposed services
- [ ] Regular system updates (`apt update && apt upgrade`)
- [ ] Documented all open ports and services

---

## What I Learned

### Technical
- How DNS actually works — and how it can be weaponized or protected
- The difference between encryption in transit vs. at rest
- How containers isolate services and why that matters for security
- Real firewall configuration vs. theoretical knowledge
- How VPNs work at a cryptographic level (not just "it hides my IP")

### Professional
- How to document infrastructure in a way others can follow
- Troubleshooting methodology: logs → isolation → testing → fix
- Reading man pages, GitHub issues, and community forums effectively
- Why security is a process, not a product

---

## Progress Tracker

| Milestone | Status | Date Completed |
|-----------|--------|----------------|
| Pi 1: OS installed & SSH configured | ✅ | March 2026 |
| Pi 1: RetroPie running | ✅ | March 2026 |
| Pi 1: WiFi connected | ✅ | March 2026 |
| Pi 1: Default password changed | ✅ | March 2026 |
| Pi 1: SSH enabled & connected | ✅ | March 2026 |
| Pi 1: System updates run | ✅ | March 2026 |
| Pi 1: SSH hardening applied | ✅ | March 2026 |
| Pi 2: Pi-hole installed & network DNS set | ⬜ | |
| Pi 2: Blocklists configured | ⬜ | |
| Pi 2: WireGuard server configured | ⬜ | |
| Pi 2: Mobile VPN client connected | ⬜ | |
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
