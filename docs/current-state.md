# 🧠 Home Server – Current State

_Last updated: 2026-04-24_

---

## 🌐 Networking

### DHCP Model
- DHCP is provided by the **Google WiFi router**
- The server operates as a **DHCP client**
- A **DHCP reservation** is configured on the router to assign:
  - `192.168.86.53` → server (via USB gigabit adapter MAC)

### Current Behavior
- Server successfully receives `192.168.86.53` via DHCP (dynamic lease)
- Reservation is now functioning correctly
- Default route is via:
  - `192.168.86.1` (router)
  - Interface: `enx6c6e072bdc14`

### Interfaces
- `enx6c6e072bdc14` (USB gigabit adapter)
  - Primary interface
  - Active and in use

- `enp1s0` (onboard ethernet)
  - Configured for DHCP
  - Physically disconnected
  - Retained as **failover / recovery interface**

### Design Decision
The onboard NIC remains configured but unplugged intentionally to allow:
- emergency physical reconnection
- recovery access without needing console/keyboard

This introduces a theoretical dual-DHCP risk if both are connected,
but is considered acceptable given controlled usage.

---

## 🕵️ Cowrie SSH Honeypot (Planned External Service)

### Overview
Cowrie is deployed as a Dockerized SSH honeypot designed to simulate a vulnerable Linux system.

It captures attacker behavior after login, including commands and session activity.

### Current Exposure State
- Running and accessible on LAN
- Bound to host port: `2222`
- Not yet exposed to the internet (no router port forwarding configured)

### Planned Exposure Model

```text
Internet TCP/22
    → Router port forward
    → 192.168.86.53:2222
    → Cowrie container (2222)
```

### Behavior
- Presents fake SSH login prompt
- Fake hostname currently set to `web-prod-nyc01`
- Uses explicit `UserDB` authentication through `userdb.txt`
- Rejects very obvious credentials such as `root/root`, `root/123456`, `root/password`, and `root/admin`
- Allows most other username/password combinations so bots can enter and interact
- Provides interactive fake shell
- Logs:
  - login attempts
  - credentials
  - commands
  - session metadata

### Data Output
Primary log file:

```text
/home/ian/cowrie/log/cowrie.json
```

Additional captured data:
- downloaded files
- full TTY session recordings

### Current Docker Volume Layout
- Cowrie config volume mounted at container path `/cowrie/cowrie-git/etc`
- Cowrie runtime volume mounted at container path `/cowrie/cowrie-git/var`
- Host bind mounts:
  - `/home/ian/cowrie/log` → `/cowrie/cowrie-git/var/log/cowrie`
  - `/home/ian/cowrie/downloads` → `/cowrie/cowrie-git/var/lib/cowrie/downloads`
  - `/home/ian/cowrie/tty` → `/cowrie/cowrie-git/var/lib/cowrie/tty`

### Security Boundary
- Real SSH service is not exposed through Cowrie
- Honeypot is isolated via Docker container
- No router-level exposure yet

---

## 🌍 DNS & Domain (Cloudflare)

### Primary Domain
- `ianboen.com`
- `www.ianboen.com`

### Additional Cloudflare Hostnames
- `jellyfin.ianboen.com`
- `cloud.ianboen.com`

### DNS Provider
- **Cloudflare (primary)**
- DuckDNS retained as **fallback / legacy**

---

## 🌐 Web Entry / Routing

### Caddy Public Routes
- `ianboen.com`, `www.ianboen.com` → Flask (`127.0.0.1:5000`)
- `jellyfin.ianboen.com` → Jellyfin (`127.0.0.1:8096`)

---

## 💽 Backup System

### Overview
The system uses a **two-layer backup architecture** with explicit directory separation:

- `/mnt/backup/storage` → data backups (user)
- `/mnt/backup/system` → system backups (root)
