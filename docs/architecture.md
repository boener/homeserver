# 🏗️ System Architecture

## 🧠 Overview

This document describes the high-level architecture of the home server.

---

## 🌐 Network Flow

Internet
→ Fiber Modem (DMZ)
→ Google WiFi Router
→ Switch
→ Server (192.168.86.53)

---

## 🔁 Request Flow

Client
→ Pi-hole (DNS)
→ Caddy (HTTPS + routing)
→ Services

---

## 🧩 Services

- Flask → port 5000
- Jellyfin → port 8096

Caddy acts as the entry point and routes traffic internally.

---

## 🔐 Security Model

- Public HTTPS via Caddy
- Internal services bound to localhost
- Jellyfin restricted to LAN via IP filtering

---

## 🧠 Key Concepts

- Reverse proxy architecture
- Split-horizon DNS
- TLS termination at proxy layer

