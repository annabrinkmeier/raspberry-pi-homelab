# Raspberry Pi Homelab

A four-Pi home lab I built to learn IT infrastructure and cybersecurity hands-on while studying for CompTIA Security+. Every service here runs on my real home network, and I document what broke along with what worked.

| Pi | Hardware | Role |
|----|----------|------|
| Pi 1 | Raspberry Pi 4 (2GB) | Nextcloud personal cloud (Docker, HTTPS) |
| Pi 2 | Raspberry Pi 5 (8GB) | Pi-hole DNS filtering, Unbound recursive DNS, WireGuard VPN |
| Pi 3 | Raspberry Pi 4 (2GB) | Jellyfin media server (Docker) with an automated disc-ripping pipeline |
| Pi 4 | Raspberry Pi 5 (8GB) | Wazuh SIEM (manager, indexer, dashboard) |

**Incident and audit write-ups**
- [DNS outage caused by a wrong system clock (November 2026)](docs/incident-2026-11-dns-clock-outage.md)
- [Audit finding: my "private" DNS resolver was forwarding to a public resolver (November 2026)](docs/finding-2026-11-unbound-forwarding.md)

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

### Security Audit (November 2026)
The firewall and Fail2ban claims above checked out. The audit found four other problems, and I fixed each one:

| Finding | Risk | Fix |
|---------|------|-----|
| Nextcloud also answered on port 8080 over plain HTTP, and Docker-published ports bypass UFW rules | Passwords could travel unencrypted, and the firewall rules did not control those ports | Recreated the container with only port 443 published. Verified 80 and 8080 are closed and HTTPS still works |
| The HTTPS setup lived only inside the container's filesystem | The next Nextcloud update or container rebuild would have silently removed HTTPS | Moved the Apache SSL config into a Dockerfile build, so the HTTPS setup is rebuilt with every update. Kept a snapshot of the old container for rollback |
| Package lists had not been refreshed since July | 41 security updates waiting, including the kernel and OpenSSL | Installed all updates with a script that stops if any package would be removed, then rebooted and re-verified every service |
| SSH allowed X11 forwarding, TCP forwarding, and key-based root login | Unneeded SSH features left on | Drop-in config disabling all three, verified with `sshd -T` and a fresh login |

### Skills Practiced
`Docker` · `Dockerfile Builds` · `SSL/TLS Certificates` · `Apache Configuration` · `Self-hosting` · `HTTPS Setup` · `OpenSSL` · `Nextcloud Administration` · `Patch Management`

### What I Learned
- How to deploy and configure Nextcloud in Docker
- How self-signed SSL certificates work and why browsers flag them
- How to configure Apache SSL inside a running Docker container — and why changes made inside a container are lost when it is recreated, unless they live in a volume or the image build
- Docker writes its own firewall rules, so a published port can be reachable even when UFW has no rule for it
- Setting up automatic photo backup from iPhone to self-hosted server
- The difference between HTTP and HTTPS and why encryption matters
- Data sovereignty — keeping personal files on your own hardware
- "0 updates available" only means something if the package list was refreshed recently

---

## Pi 2 — Pi-hole + WireGuard VPN

### Purpose
Transform a Raspberry Pi into a network-wide ad blocker, personal VPN server, intrusion prevention system, and private recursive DNS resolver.

### Pi-hole Configuration
- Installed Pi-hole v6 via official installer
- Set as network-wide DNS via router (Plume), so every device on the network is covered automatically
- Blocklists: StevenBlack + HaGeZi Pro, updated automatically. In November 2026 HaGeZi retired the hosts-format file I used; I switched to HaGeZi's adblock-format Pro list, where one entry blocks a domain and all its subdomains (about 318,000 entries in total)
- Blocks roughly 27–29% of all DNS lookups on the network
- Whitelisted Apple iCloud Private Relay to maintain iPhone functionality
- Identified and blocked Roku TV surveillance (Alphonso.tv), Amazon Alexa telemetry, Sift Science behavioral tracking, and Datadog browser monitoring
- Reviewed the query log per device in November 2026 and added custom deny entries for LG smart TV data-collection servers and a Microsoft telemetry endpoint used by the Xbox and gaming PC
- Configured static IP via NetworkManager (nmcli)

### Unbound Recursive DNS
- Installed Unbound as local recursive DNS resolver
- Pi-hole sends lookups to Unbound, which queries the root DNS servers directly instead of handing my browsing history to one public resolver
- DNS answers are cached locally, so repeat lookups return almost instantly
- DNSSEC validation enabled and tested (a deliberately broken test domain is refused)
- **Correction (November 2026):** an audit showed Unbound had been quietly forwarding every lookup to a public resolver, so this setup was less private than I had claimed here. My first fix covered only one of two causes; a reboot test exposed the second. Both are fixed now, and the fix has been verified across a reboot. [Full write-up](docs/finding-2026-11-unbound-forwarding.md)

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

### Security Hardening Session (September 2026)
- Fixed 19 additional CIS benchmark checks in one session
- CIS score progression: 25% → 40% → 49% (nearly doubled from start)
- Installed AIDE file integrity monitoring with a daily check via systemd timer
- Set up auditd with custom rules monitoring critical system files
- Configured password complexity requirements (14 character minimum)
- Implemented login lockout policy (5 attempts = 10 minute lockout)
- Hardened SSH configuration (MaxAuthTries, legal warning banner)
- Applied kernel security parameters (ASLR, core dumps, SYN cookies, ICMP redirects, source routing)
- Fixed file permissions on /etc/passwd, /etc/shadow, /etc/group
- Configured NTP time synchronization with chrony
- Fixed cron and at command permissions
- Set noexec on /dev/shm to prevent executable code in shared memory
- Disabled USB storage module

### Security Audit (November 2026)
I audited this Pi against what this README claimed. Four things did not match, and I fixed each one:

| Finding | Risk | Fix |
|---------|------|-----|
| System clock was 55 days behind after a reboot, which made every DNS lookup fail | Network-wide DNS outage | Set the clock, gave chrony time servers by IP address so time sync no longer depends on DNS. [Write-up](docs/incident-2026-11-dns-clock-outage.md) |
| UFW firewall package was no longer installed | No host firewall on the DNS/VPN server | Reinstalled UFW with default-deny incoming and only the needed ports (SSH, DNS, web dashboard, WireGuard). Enabled it with an automatic rollback timer so a mistake could not lock me out |
| Unbound was forwarding lookups to a public resolver | Privacy claim in this README was wrong | Turned off both mechanisms that added the forwarders (a systemd helper and a resolvconf hook), verified direct recursion and DNSSEC, and confirmed the fix survives a reboot. [Write-up](docs/finding-2026-11-unbound-forwarding.md) |
| HaGeZi Pro blocklist URL started returning 404 | Blocking quietly fell back to one list during a routine update | Switched to HaGeZi's current adblock-format Pro list and rebuilt the blocklist |
| SSH X11 and TCP forwarding were still enabled | Unneeded SSH features left on | Added a drop-in config file disabling both, verified with `sshd -T` |

I also removed a leftover lighttpd web server that Pi-hole v6 no longer needs, and installed 155 system updates plus the latest Pi-hole release (backups first, then a reboot and a full re-check).

### Skills Practiced
`DNS Administration` · `Network-wide Filtering` · `VPN Configuration` · `Public Key Cryptography` · `Firewall Rules (UFW/iptables)` · `IP Forwarding` · `Port Forwarding` · `Intrusion Prevention` · `Recursive DNS` · `DNSSEC` · `DNS Privacy` · `Unbound` · `Security Auditing` · `Incident Response` · `Time Synchronization (NTP)`

### What I Learned
- How DNS sinkholing works at a network level
- DNS single points of failure — configured 1.1.1.1 as fallback DNS
- Pi-hole v6 completely changed the web interface — had to troubleshoot using curl and systemctl
- IP forwarding resets to 0 on reboot unless saved to `/etc/sysctl.d/`
- A single character typo (`erth0` instead of `eth0`) broke VPN routing for months — fixed by carefully auditing config files character by character
- Difference between IDS (detection) and IPS (prevention) — Fail2ban is active prevention
- Pi-hole cannot block YouTube ads because Google serves ads from the same domains as video content
- Smart devices (Alexa, Roku, smart TVs) constantly send telemetry — Pi-hole blocks surveillance while maintaining functionality
- Difference between recursive and iterative DNS resolution
- DNSSEC validates DNS responses haven't been tampered with — and it depends on an accurate clock
- ISP DNS privacy risks and how to mitigate them
- A Raspberry Pi has no battery-backed clock, so a reboot without working time sync can break anything that checks dates and signatures
- Installing a control once is different from verifying it is still in place — configuration drifts, so audits need to be repeated

---

## Pi 3 — Pi Movie Vault

### Purpose
Self-hosted media server using Docker on a Raspberry Pi 4.

### Completed
**Jellyfin Media Server** ✅
- Installed via Docker on Raspberry Pi OS Lite (64-bit)
- Connected 3.6TB external SSD for media storage
- Configured auto-mount via fstab (exFAT format)
- Accessible on local network via port 8096
- Streaming to Apple TV via Swiftfin app
- Separate TV and Movies libraries, with Docker volume mounts that expose only the finished show and movie folders to Jellyfin, so posters, episode titles, and descriptions are fetched automatically
- UFW firewall configured
- Fail2ban for SSH (reinstalled November 2026 — an audit found it missing after an SD card swap)
- Wazuh agent connected for SIEM monitoring

**Automated disc-image ripping pipeline** 🔄 (November 2026)
- Converting 116 episodes and movies from DVD disc images to MKV, unattended, over about three days (in progress)
- Each existing file was matched to its disc title by comparing chapter lengths with `ffprobe`, so episodes were identified by evidence instead of by guessing
- A plain-text queue file lists one job per line: disc image, title number, output path
- A Bash script works through the queue with HandBrakeCLI, writes to a temporary file, and moves it into place only when the encode succeeds
- Finished episodes are skipped on a rerun, so the queue can be restarted safely
- Every start, finish, and failure is logged with a timestamp and the CPU temperature
- Runs detached with `nohup`, so it continues after the SSH session ends

### Skills Practiced
`Docker` · `Volume Mounts` · `Bash Scripting` · `Job Automation` · `Logging` · `HandBrakeCLI` · `ffprobe` · `Filesystem Mounting (fstab)`

### What I Learned
- Docker installation and container management
- Volume mounts and persistent storage in Docker
- exFAT filesystem mounting and fstab configuration
- Hotel WiFi uses client isolation — devices on the same network can't communicate
- Ethernet vs WiFi for media streaming reliability
- DVD ISO extraction and MKV conversion workflow
- Media servers depend on consistent file naming (`Show - S01E01 - Title`) to match episodes to metadata
- Writing to a temporary file and renaming on success keeps half-finished files out of the library
- Rename by moving, and count files and bytes before and after, when the source media cannot be replaced

---

## Pi 4 — Security Operations Center

### Purpose
Dedicated Wazuh SIEM server to collect, correlate, and analyze security logs from all other Pis — building a real Security Operations Center.

### Current Status: ✅ Operational

### Completed
- Wazuh Manager installed and running
- Wazuh Indexer configured with SSL certificates
- Wazuh Dashboard accessible via HTTPS
- 3 agents connected (Pi-hole, Jellyfin, Nextcloud)
- CIS Debian 13 benchmark running on all agents — first run: 74 passed, 108 failed (25/100 score, normal for fresh install)
- UFW firewall configured (ports 22, 443, 1514, 1515)
- Fail2ban installed and configured
- SSL certificates manually generated for all Wazuh components

### Security Audit (November 2026)
The firewall and Fail2ban claims above checked out. Findings and fixes:

| Finding | Risk | Fix |
|---------|------|-----|
| Wazuh's database (ports 9200/9300) and admin API (port 55000) listen on every network address | Would be exposed without the host firewall | Verified UFW blocks all three. No change needed — this is the firewall doing its job |
| Wazuh packages were not version-locked, and a newer release was available | A routine update could upgrade the manager or an agent out of step; agents newer than the manager are not supported | Locked the manager, indexer, dashboard, and every agent with `apt-mark hold`. Wazuh upgrades are now a planned job |
| Package lists had not been refreshed since May | 43 security updates waiting, including the kernel and OpenSSL | Installed all updates, rebooted, confirmed all three agents reconnected |
| SSH allowed X11 forwarding, TCP forwarding, and key-based root login | Unneeded SSH features left on | Drop-in config disabling all three |
| Pis were set to different time zones | Events from different machines are harder to line up in a SIEM timeline | Set all four Pis to the same time zone |

### Skills Practiced
`SIEM Deployment` · `Log Analysis` · `Security Monitoring` · `CIS Benchmarks` · `SSL Certificate Management` · `ARM Architecture Deployment` · `OpenSearch Configuration` · `Patch Management` · `Change Control`

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
| Pi 2: CIS benchmark score improved 25% → 49% | ✅ | September 2026 |
| Pi 2: Kernel hardening parameters configured | ✅ | September 2026 |
| Pi 2: File permissions hardened | ✅ | September 2026 |
| Pi 2: AIDE file integrity monitoring installed | ✅ | September 2026 |
| Pi 2: Password complexity requirements configured | ✅ | September 2026 |
| Pi 2: Login lockout policy configured | ✅ | September 2026 |
| Pi 2: SSH hardening configured | ✅ | September 2026 |
| Pi 2: Cron permissions hardened | ✅ | September 2026 |
| Pi 2: Auditd logging installed | ✅ | September 2026 |
| Pi 2: Audit rules configured | ✅ | September 2026 |
| Pi 2: Security audit against this README | ✅ | November 2026 |
| Pi 2: DNS outage diagnosed and fixed (clock / DNSSEC) | ✅ | November 2026 |
| Pi 2: UFW firewall found missing and restored | ✅ | November 2026 |
| Pi 2: Unbound forwarding removed, direct recursion verified across a reboot | ✅ | November 2026 |
| Pi 2: System and Pi-hole updates installed, blocklist source repaired | ✅ | November 2026 |
| Pi 2: SSH X11 and TCP forwarding disabled | ✅ | November 2026 |
| Pi 2: Per-device query log review, custom deny entries | ✅ | November 2026 |
| Pi 3: OS installed & SSH configured | ✅ | April 2026 |
| Pi 3: Docker installed | ✅ | April 2026 |
| Pi 3: Jellyfin installed via Docker | ✅ | April 2026 |
| Pi 3: Media SSD mounted (3.6TB) | ✅ | April 2026 |
| Pi 3: Jellyfin accessible on Apple TV | ✅ | April 2026 |
| Pi 3: SSD auto-mount configured | ✅ | April 2026 |
| Pi 3: UFW firewall configured | ✅ | May 2026 |
| Pi 3: Fail2ban installed | ✅ | May 2026 |
| Pi 3: Wazuh agent connected | ✅ | May 2026 |
| Pi 3: Homarr dashboard deployed | ✅ | August 2026 |
| Pi 3: Fail2ban found missing and reinstalled | ✅ | November 2026 |
| Pi 3: Episodes identified by chapter fingerprint and renamed | ✅ | November 2026 |
| Pi 3: Automated ripping pipeline built and running (116 titles) | 🔄 | |
| Pi 3: Jellyfin TV and Movies libraries with metadata | ✅ | November 2026 |
| Pi 4: Wazuh SIEM deployed | ✅ | May 2026 |
| Pi 4: Wazuh Indexer configured | ✅ | May 2026 |
| Pi 4: Wazuh Dashboard accessible | ✅ | May 2026 |
| Pi 4: UFW firewall configured | ✅ | May 2026 |
| Pi 4: Fail2ban installed | ✅ | May 2026 |
| Pi 4: 3 agents reporting | ✅ | May 2026 |
| Pi 4: CIS Benchmark audit running | ✅ | May 2026 |
| Pi 4: Wazuh password reset and secured | ✅ | September 2026 |
| Pi 1: Security audit, plain-HTTP port closed, HTTPS made update-proof | ✅ | November 2026 |
| Pi 1: Security updates installed (41) | ✅ | November 2026 |
| Pi 4: Security audit, Wazuh packages version-locked | ✅ | November 2026 |
| Pi 4: Security updates installed (43) | ✅ | November 2026 |
| All: SSH key-based auth on all Pis | ✅ | March 2026 |
| All: Wazuh monitoring on all Pis | ✅ | August 2026 |
| All: Homelab running unattended for 7 weeks | ✅ | September 2026 |
| Full security audit: Pi 3 | 🔄 | |
| Documentation complete | 🔄 | |

✅ Complete · 🔄 In Progress · ⬜ Not Started

---

## Future Plans

### Security Hardening
- [x] Repeat the README-versus-reality audit on Pi 1 and Pi 4
- [ ] Repeat the README-versus-reality audit on Pi 3
- [ ] Plan the Wazuh upgrade (manager, indexer, dashboard, then agents)
- [ ] Continue CIS benchmark fixes on Pi 2 (currently 49%) and start on the other Pis
- [ ] Add a real-time clock battery to the Pi 5 units so the clock survives power loss
- [ ] Enable 2FA on Nextcloud
- [ ] Enable 2FA on Wazuh dashboard
- [ ] Upgrade Nextcloud from SQLite to MariaDB
- [ ] Configure automatic security updates on all Pis
- [ ] Set up 3-2-1 backup strategy

### Infrastructure
- [x] Homarr unified dashboard — one page for all services
- [ ] Finish Sopranos/Yellowstone MKV conversion (automated queue running)
- [ ] Set up Nextcloud desktop sync on Mac
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
- Security audit and incident write-ups (Domain 4 — Incident Response)

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

Built and maintained by Anna Brinkmeier as part of a self-directed IT and cybersecurity learning journey. I use AI assistants as a tutor and pair-administrator, and I review and approve each change made to my systems.

🔗 [LinkedIn](https://www.linkedin.com/in/anna-brinkmeier-246b34345)
🐙 [GitHub](https://github.com/annabrinkmeier)

---

*"The best way to learn infrastructure is to build it, break it, and document why it broke."*
