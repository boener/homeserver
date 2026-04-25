# 🧠 Home Server – Current State

_Last updated: 2026-04-25_

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

- `/mnt/backup/storage`
- `/mnt/backup/system`
