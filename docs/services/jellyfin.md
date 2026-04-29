# 🎬 Jellyfin Service

## 🧠 Overview

Jellyfin provides media streaming from the home server.

---

## ⚙️ Configuration

- Port: 8096
- Accessed via: https://jellyfin.ianboen.com and https://jellyboen.duckdns.org (retiring)
- Reverse proxied through Caddy

---

## 🔐 Access Control

- https://jellyboen.duckdns.org (retiring)
- Allowed: LAN (192.168.x.x)
- Blocked: External traffic (403 via Caddy)

- https://jellyfin.ianboen.com (Allowed)

---

## 💾 Media

- Location: /mnt/storage

---

## 🧠 Notes

- Hardware transcoding enabled
- Works with Roku and browser clients

