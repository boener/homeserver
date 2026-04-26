# 🧠 Home Server – Current State

_Last updated: 2026-04-26_

---

## 🖥️ Nodes

### Primary Server (`ubuntu`)

- Role: main application host
- IP: `192.168.86.53`
- Hosts:
  - Caddy (web entry)
  - Flask app
  - Jellyfin
  - Cowrie honeypot
  - Backup system
  - Outbound email (Postfix + SMTP2Go)

### Secondary Node (`macbook`)

- Role: lab / auxiliary node
- IP: `192.168.86.45`
- Documentation:
  - `docs/nodes/macbook.md`

---

## 🌐 Networking

### DHCP Model
- DHCP is provided by the **Google WiFi router**
- All nodes operate as **DHCP clients**
- DHCP reservations assign stable IPs

### Primary Server Interface
- `enx6c6e072bdc14` (USB gigabit adapter)

### Secondary Node Interface
- `enp2s0f0` (MacBook onboard ethernet)

### Default Gateway
- `192.168.86.1`

---

## 🕵️ Cowrie SSH Honeypot (Primary Server)

### Current Exposure State
- Running and accessible on LAN
- Bound to host port: `2222`
- Not yet exposed to the internet

---

## 📧 Outbound Email (Primary Server)

### Status
- Operational
- Uses SMTP2Go relay

### Function
- Sends system alerts and notifications
- Used by scripts, cron jobs, and services

See: `docs/services/outbound-email.md`

---

## 🌍 DNS & Domain (Cloudflare)

### Primary Domain
- `ianboen.com`
- `www.ianboen.com`

### Additional Hostnames
- `jellyfin.ianboen.com`
- `cloud.ianboen.com`

---

## 🌐 Web Entry / Routing

### Caddy Public Routes (Primary Server)
- `ianboen.com`, `www.ianboen.com` → Flask (`127.0.0.1:5000`)
- `jellyfin.ianboen.com` → Jellyfin (`127.0.0.1:8096`)

---

## 💽 Backup System (Primary Server)

### Mount Point
- `/mnt/backup`

### Subdirectories
- `/mnt/backup/storage`
- `/mnt/backup/system`

### Mount Behavior
- External USB drive mounted by UUID
- Uses `nofail` and `x-systemd.automount`
- Mounted on first access rather than at boot

### Backup Execution
- Storage backup:
  - Script: `/home/ian/backup.sh`
  - Schedule: `0 3 * * *` (user cron)

- System backup:
  - Script: `/usr/local/sbin/system-backup.sh`
  - Schedule: `30 3 * * *` (root cron)

See `docs/services/backup.md` for full details.
