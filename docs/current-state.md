# 🧠 Home Server – Current State

_Last updated: 2026-04-19_

---

## 🖥️ Hardware

- Dell Inspiron 5570 laptop
- i5-8250U (4c/8t)
- 16GB RAM
- 512GB SSD (OS)
- 1TB HDD (/mnt/storage)
- USB 3.0 Gigabit NIC (primary)

---

## 🌐 Network Architecture

Internet → Fiber Modem (DMZ) → Google WiFi → Switch → Server (192.168.86.53)

- Pi-hole: 192.168.86.49 (DNS)
- Google WiFi handles DHCP + routing
- Ports 80/443 forwarded to server

---

## 🔐 Reverse Proxy (Caddy)

- TLS via Let’s Encrypt
- Routes:
  - boener.duckdns.org → Flask (:5000)
  - jellyboen.duckdns.org → Jellyfin (:8096)
- Jellyfin restricted to LAN

---

## 🎬 Jellyfin

- Port: 8096
- Local HTTPS via Caddy
- Media: /mnt/storage
- Hardware transcoding enabled

---

## 🐍 Flask

- Port: 5000
- Systemd service

---

## 💾 Storage

- /mnt/storage (ext4)
- Samba shares enabled

---

## ⚠️ Risks

- No backups
- No redundancy
- Single disk

---

## 🧾 Changelog

### 2026-04-19
- Established baseline documentation
- USB gigabit adapter now primary network interface
- Jellyfin + Caddy + HTTPS fully working
