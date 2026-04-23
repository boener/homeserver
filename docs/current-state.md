# 🧠 Home Server – Current State

_Last updated: 2026-04-22_

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

### Previous Issue (Resolved)
- Router previously issued incorrect IP (`.22`) despite reservation
- Likely caused by:
  - stale DHCP lease
  - interface/MAC mismatch
- Temporary workaround used:
  - static IP assignment on server

### Resolution
- Re-enabled DHCP on server
- Router now correctly honors reservation
- Static configuration removed

---

## 🌍 DNS & Domain (Cloudflare)

### Primary Domain
- `ianboen.com`
- `www.ianboen.com`

### DNS Provider
- **Cloudflare (primary)**
- DuckDNS retained as **fallback / legacy**

### Dynamic DNS (Cloudflare)

The system uses a custom dynamic DNS updater:

- Script:
  - `/home/ian/cloudflare-ddns.sh`

- Config:
  - `~/.cloudflare/ddns.env`
  - contains:
    - API token
    - Zone ID
    - DNS record IDs

### Behavior
- Script fetches current public IP via:
  - `curl ifconfig.me`
- Updates both:
  - root domain (`ianboen.com`)
  - `www` subdomain

### Automation
*/5 * * * * /home/ian/cloudflare-ddns.sh >/dev/null 2>&1
- Runs every 5 minutes
- Silent execution (no cron spam)

### Design Notes
- Cloudflare replaces DuckDNS as the **authoritative DNS provider**
- Dynamic DNS is now fully self-managed
- Record-level updates avoid unnecessary API calls to unrelated entries

### DuckDNS Status
- Still configured and functional
- Used for:
  - fallback access
  - testing
  - redundancy experiments
- Not part of primary routing path

---

## 💽 Backup System

### Overview
The system uses a **two-layer backup architecture** with explicit directory separation:

- `/mnt/backup/storage` → data backups (user)
- `/mnt/backup/system` → system backups (root)

This prevents accidental cross-deletion and keeps responsibilities clearly isolated.

---

## 📦 Data Backup (User)

### Scope
- `/mnt/storage`

### Ownership
- Runs as user `ian`

### Structure
```
/mnt/backup/storage/
├── current/
└── snapshots/YYYY-MM-DD/
```

### Script
- `/home/ian/backup.sh`

### Behavior
- `rsync -a --delete` syncs data into `current/`
- `cp -al` creates snapshot using hard links
- retention: ~14 days

### Automation
```
0 3 * * * /home/ian/backup.sh
```

---

## ⚙️ System Backup (Root)

### Scope
- `/etc`
- `/home/ian`

### Ownership
- Runs as `root`

### Structure
```
/mnt/backup/system/
├── current/
│   ├── etc/
│   └── home-ian/
└── snapshots/YYYY-MM-DD/
```

### Script
- `/usr/local/sbin/system-backup.sh`

### Behavior
- `rsync -a --delete` for system files
- snapshot via `cp -al`
- replaces same-day snapshot if rerun
- retention: ~14 days

### Automation
```
30 3 * * * /usr/local/sbin/system-backup.sh
```

---

## 🧠 Design Notes

### Why the `storage/` subdirectory was introduced
Originally, data backups lived directly under `/mnt/backup/`, which made it easy to:
- accidentally delete the entire backup tree
- mix system and data backup concerns

Moving to:
```
/mnt/backup/storage/
```
creates a clear boundary and reduces risk.

### Failure Mode Prevented
- accidental deletion of `/mnt/backup/*` no longer destroys both backup systems
- system backups remain intact even if storage backup is misconfigured

---

## 🧾 Changelog

### 2026-04-20
- Refined backup structure to separate `/mnt/backup/storage` and `/mnt/backup/system`
- Updated storage backup script to use new path
- Improved safety against accidental deletion
