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
- Hosts:
  - WireGuard VPN
- Service documentation:
  - `docs/services/wireguard-vpn.md`

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

## 🔐 WireGuard VPN (Secondary Node)

### Status
- Operational
- Runs on `macbook`
- Enabled at boot and reboot-tested

### Network Details
- Host LAN IP: `192.168.86.45`
- LAN interface: `enp2s0f0`
- VPN interface: `wg0`
- VPN subnet: `10.0.0.0/24`
- Server VPN IP: `10.0.0.1`
- First client VPN IP: `10.0.0.2`
- Public port: UDP `51820`

### Router Forward
- External UDP `51820` → `192.168.86.45:51820`

### DNS
- VPN clients use Pi-hole: `192.168.86.49`

### Endpoint
- `ianboen.com:51820`
- Cloudflare DNS record must be DNS-only, not proxied

See: `docs/services/wireguard-vpn.md`

## 🕵️ Cowrie SSH Honeypot (Primary Server)

### Current Exposure State
- Running and accessible
- Bound to host port: `2222`
- Exposed to the internet: Router forwards external port `22` to host port `2222`
- Unaffected by WireGuard; Cowrie remains on external TCP `22`, while WireGuard uses UDP `51820`

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
- `cloud.ianboen.com` (planned)

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
