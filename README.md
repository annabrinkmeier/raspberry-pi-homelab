---

## Pi 1 — Nextcloud Personal Cloud

### Purpose
Self-hosted personal cloud storage — covering Docker deployment, HTTPS/SSL configuration, and self-hosted services. Replaced RetroPie gaming console to build more relevant IT portfolio skills.

### What I Did
- Reflashed Pi 4 2GB with Raspberry Pi OS Lite (64-bit)
- Installed Docker
- Deployed Nextcloud via Docker container
- Generated self-signed SSL certificate with OpenSSL
- Configured Apache SSL inside Docker container
- Set up HTTPS on port 443
- Configured iPhone auto upload for automatic photo backup
- Configured UFW firewall (ports 22, 443)
- Installed Fail2ban for SSH brute force protection
- Connected Wazuh agent for SIEM monitoring
- Added trusted domains for local network access

### Skills Practiced
`Docker` · `SSL/TLS Certificates` · `Apache Configuration` · `Self-hosting` · `HTTPS Setup` · `OpenSSL` · `Nextcloud Administration`

### What I Learned
- How to deploy and configure Nextcloud in Docker
- How self-signed SSL certificates work and why browsers flag them
- How to configure Apache SSL inside a running Docker container
- Setting up automatic photo backup from iPhone to self-hosted server
- The difference between HTTP and HTTPS and why encryption matters
- Data sovereignty — keeping personal files on your own hardware

---

## Pi 2 — Pi-hole + WireGuard VPN

### Purpose
Transform a Raspberry Pi into a network-wide ad blocker, personal VPN server, intrusion prevention system, and private recursive DNS resolver.

### Pi-hole Configuration
- Installed Pi-hole v6 via official installer
- Set as network-wide DNS via router (Plume) — **28 active clients** protected automatically
- Blocking **607,158 domains** (StevenBlack + HaGeZi Pro blocklists) — auto-updates while running unattended
- Currently blocking **~26.7% of all network traffic**
- Whitelisted Apple iCloud Private Relay to maintain iPhone functionality
- Identified and blocked Roku TV surveillance (Alphonso.tv), Amazon Alexa telemetry, Sift Science behavioral tracking, and Datadog browser monitoring
- Configured static IP via NetworkManager (nmcli)
- Pi-hole has maintained 7+ weeks of unattended uptime with automatic blocklist updates
- Fixed CIS benchmark failures:
- Set noexec on /dev/shm to prevent executable code in shared memory
- Hardened kernel parameters (SYN cookies, ICMP redirects, source routing)
- Fixed file permissions on /etc/passwd, /etc/shadow, /etc/group
- CIS benchmark score improved from 25% to 40%
- Installed AIDE file integrity monitoring — detects unauthorized file changes
- Configured daily AIDE filesystem checks via systemd timer
- Hardened password quality requirements (14 char min, complexity rules)
- Configured login attempt lockout (5 attempts = 10 min lockout)
- Set noexec on /dev/shm to prevent malicious code execution


### Unbound Recursive DNS
- Installed Unbound as local recursive DNS resolver
- Pi-hole now queries root DNS servers directly — bypasses ISP and Google DNS completely
- DNS queries cached locally — 0ms response time after first lookup
- DNSSEC validation enabled

### WireGuard VPN Configuration
- Generated public/private keypairs for server and clients using Curve25519
- Configured `wg0.conf` interface and peer definitions from scratch
- Set up IP forwarding (`net.ipv4.ip_forward=1`) — made permanent via `/etc/sysctl.d/`
- Configured UFW and iptables firewall rules
- Set up port forwarding on router
- Connected iOS client via QR code
- Configured on-demand activation — auto-connects on cellular, auto-disconnects on home WiFi

### Fail2ban Configuration
- Installed and configured Fail2ban for SSH intrusion prevention
- Custom ban policies configured
- Actively monitoring SSH login attempts

### Skills Practiced
`DNS Administration` · `Network-wide Filtering` · `VPN Configuration` · `Public Key Cryptography` · `Firewall Rules (UFW/iptables)` · `IP Forwarding` · `Port Forwarding` · `Intrusion Prevention` · `Recursive DNS` · `DNSSEC` · `DNS Privacy` · `Unbound`

### What I Learned
- How DNS sinkholing works at a network level
- DNS single points of failure — configured 1.1.1.1 as fallback DNS
- Pi-hole v6 completely changed the web interface — had to troubleshoot using curl and systemctl
- IP forwarding resets to 0 on reboot unless saved to `/etc/sysctl.d/`
- A single character typo (`erth0` instead of `eth0`) broke VPN routing for months — fixed by carefully auditing config files character by character
- Difference between IDS (detection) and IPS (prevention) — Fail2ban is active prevention
- Pi-hole cannot block YouTube ads because Google serves ads from the same domains as video content
- Smart devices (Alexa, Roku) constantly send telemetry — Pi-hole blocks surveillance while maintaining functionality
- Difference between recursive and iterative DNS resolution
- DNSSEC validates DNS responses haven't been tampered with
- ISP DNS privacy risks and how to mitigate them

---

## Pi 3 — Pi Movie Vault

### Purpose
Self-hosted media server using Docker on a Raspberry Pi 4.

### Current Status: Jellyfin Complete ✅

### Completed
**Jellyfin Media Server** ✅
- Installed via Docker on Raspberry Pi OS Lite (64-bit)
- Connected 3.6TB external SSD for media storage
- Configured auto-mount via fstab (exFAT format)
- Accessible on local network via port 8096
- Streaming to Apple TV via Swiftfin app
- Converting ISO disc images to MKV using MakeMKV and FileBot for proper episode organization
- Connected via ethernet for stable streaming
- UFW firewall configured
- Wazuh agent connected for SIEM monitoring

### What I Learned
- Docker installation and container management
- Volume mounts and persistent storage in Docker
- exFAT filesystem mounting and fstab configuration
- Hotel WiFi uses client isolation — devices on the same network can't communicate
- Ethernet vs WiFi for media streaming reliability
- DVD ISO extraction and MKV conversion workflow

---

## Pi 4 — Security Operations Center

### Purpose
Dedicated Wazuh SIEM server to collect, correlate, and analyze security logs from all other Pis — building a real Security Operations Center.

### Current Status: ✅ Fully Operational

### Completed
- Wazuh Manager installed and running (5+ days uptime)
- Wazuh Indexer configured with SSL certificates
- Wazuh Dashboard accessible via HTTPS
- 3 agents connected (Pi-hole, Jellyfin, Nextcloud)
- CIS Debian 13 benchmark running on all agents — 74 passed, 108 failed (25/100 score, normal for fresh install)
- UFW firewall configured (ports 22, 443, 1514, 1515)
- Fail2ban installed and configured
- SSL certificates manually generated for all Wazuh components

### Skills Practiced
`SIEM Deployment` · `Log Analysis` · `Security Monitoring` · `CIS Benchmarks` · `SSL Certificate Management` · `ARM Architecture Deployment` · `OpenSearch Configuration`

### What I Learned
- Deployed Wazuh SIEM manually on ARM architecture (official installer only supports x86_64)
- Manually generated SSL certificates for Wazuh Indexer, Manager, and Dashboard
- Connected agents across multiple Pis using ports 1514/1515
- Ran CIS Debian 13 benchmark — understanding security hardening standards
- SIEM architecture: Manager collects, Indexer stores, Dashboard visualizes
- How enterprise SOC tools like Splunk and QRadar work at a conceptual level
- MITRE ATT&CK framework integration in Wazuh

---

## Progress Tracker

| Milestone | Status | Date Completed |
|-----------|--------|----------------|
| Pi 1: OS installed & SSH configured | ✅ | March 2026 |
| Pi 1: RetroPie running | ✅ | March 2026 |
| Pi 1: SSH hardening applied | ✅ | March 2026 |
| Pi 1: UFW firewall configured | ✅ | March 2026 |
| Pi 1: Converted to Nextcloud | ✅ | July 2026 |
| Pi 1: Docker installed | ✅ | July 2026 |
| Pi 1: Nextcloud running via Docker | ✅ | July 2026 |
| Pi 1: HTTPS/SSL configured | ✅ | August 2026 |
| Pi 1: iPhone auto upload configured | ✅ | August 2026 |
| Pi 1: Fail2ban installed | ✅ | August 2026 |
| Pi 1: Wazuh agent connected | ✅ | August 2026 |
| Pi 2: OS installed & SSH configured | ✅ | March 2026 |
| Pi 2: Pi-hole installed & network DNS set | ✅ | March 2026 |
| Pi 2: Blocklists configured (543k domains) | ✅ | March 2026 |
| Pi 2: UFW firewall configured | ✅ | March 2026 |
| Pi 2: Web dashboard accessible | ✅ | March 2026 |
| Pi 2: WireGuard server configured | ✅ | March 2026 |
| Pi 2: WireGuard client configured | ✅ | March 2026 |
| Pi 2: Port forwarding configured | ✅ | April 2026 |
| Pi 2: Mobile VPN client connected | ✅ | April 2026 |
| Pi 2: VPN routing fixed (eth0 typo) | ✅ | May 2026 |
| Pi 2: Pi-hole static IP configured | ✅ | May 2026 |
| Pi 2: Network-wide DNS via Plume configured | ✅ | May 2026 |
| Pi 2: 20+ devices protected by Pi-hole | ✅ | May 2026 |
| Pi 2: WireGuard on-demand cellular configured | ✅ | May 2026 |
| Pi 2: Fail2ban installed and configured | ✅ | May 2026 |
| Pi 2: Unbound recursive DNS installed | ✅ | May 2026 |
| Pi 2: CIS benchmark score improved 25% → 40% | ✅ | September 2026 |
| Pi 2: Kernel hardening parameters configured | ✅ | September 2026 |
| Pi 2: File permissions hardened | ✅ | September 2026 |
| Pi 2: AIDE file integrity monitoring installed | ✅ | September 2026 |
| Pi 2: Password complexity requirements configured | ✅ | September 2026 |
| Pi 2: Login lockout policy configured | ✅ | September 2026 |
| Pi 2: SSH hardening configured | ✅ | September 2026 |
| Pi 2: Cron permissions hardened | ✅ | September 2026 |
| Pi 2: Auditd logging installed | ✅ | September 2026 |
| Pi 2: Audit rules configured | ✅ | September 2026 |
| Pi 3: OS installed & SSH configured | ✅ | April 2026 |
| Pi 3: Docker installed | ✅ | April 2026 |
| Pi 3: Jellyfin installed via Docker | ✅ | April 2026 |
| Pi 3: Media SSD mounted (3.6TB) | ✅ | April 2026 |
| Pi 3: Jellyfin accessible on Apple TV | ✅ | April 2026 |
| Pi 3: SSD auto-mount configured | ✅ | April 2026 |
| Pi 3: UFW firewall configured | ✅ | May 2026 |
| Pi 3: Fail2ban installed | ✅ | May 2026 |
| Pi 3: Wazuh agent connected | ✅ | May 2026 |
| Pi 4: Wazuh SIEM deployed | ✅ | May 2026 |
| Pi 4: Wazuh Indexer configured | ✅ | May 2026 |
| Pi 4: Wazuh Dashboard accessible | ✅ | May 2026 |
| Pi 4: UFW firewall configured | ✅ | May 2026 |
| Pi 4: Fail2ban installed | ✅ | May 2026 |
| Pi 4: 3 agents reporting | ✅ | May 2026 |
| Pi 4: CIS Benchmark audit running | ✅ | May 2026 |
| All: SSH key-based auth on all Pis | ✅ | March 2026 |
| All: Fail2ban on all Pis | ✅ | May 2026 |
| All: UFW firewall on all Pis | ✅ | May 2026 |
| All: Wazuh monitoring on all Pis | ✅ | August 2026 |
| Documentation complete | 🔄 | |
| All: Homelab running unattended for 7 weeks | ✅ | September 2026 |
| Pi 4: Wazuh password reset and secured | ✅ | September 2026 |

✅ Complete · 🔄 In Progress · ⬜ Not Started

---

## Future Plans

### Security Hardening
- [ ] Fix CIS benchmark failures (improve from 25/100)
- [ ] Enable 2FA on Nextcloud
- [ ] Enable 2FA on Wazuh dashboard
- [ ] Upgrade Nextcloud from SQLite to MariaDB
- [ ] Configure automatic security updates on all Pis
- [ ] Set up 3-2-1 backup strategy

### Infrastructure
- [ ] Homarr unified dashboard — one page for all services
- [ ] Set up Nextcloud desktop sync on Mac
- [ ] Finish Sopranos/Yellowstone MKV conversion
- [ ] Add honeypot to observe real attack patterns
- [ ] Configure centralized logging across all Pis
- [ ] Set up Grafana + Prometheus monitoring dashboards

### Security+ Lab Roadmap
Structured additions aligned with Security+ exam domains:

**Current ✅**
- Pi-hole DNS sinkhole (Domain 2 — Threat Intelligence)
- Unbound recursive DNS (Domain 3 — Network Security)
- WireGuard VPN (Domain 3 — Network Security)
- Nextcloud in Docker (Domain 3 — Containerization)
- UFW Firewall on all Pis (Domain 3 — Firewall Rules)
- SSH Hardening + Key-based Auth (Domain 1 — Authentication)
- Fail2ban IPS (Domain 2 — Intrusion Prevention)
- Wazuh SIEM (Domain 4 — Security Operations)
- CIS Benchmarks (Domain 4 — Security Assessment)
- SSL/TLS Certificates (Domain 3 — Cryptography)

**After Lesson 3 ⬜**
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
