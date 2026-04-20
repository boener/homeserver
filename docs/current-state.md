# 🧠 Home Server – Current State

_Last updated: 2026-04-19_

---

## 🧠 Purpose of This Document

This file is the **canonical, current-state snapshot** of the home server.

It is intentionally:
- Detailed (so concepts are preserved)
- Accurate (reflects real system state)
- Updated over time (acts as living memory)

---

## 🖥️ Hardware

### System
- Dell Inspiron 5570 laptop
- Intel i5-8250U (4 cores / 8 threads)
- 16GB RAM

### Storage
- 512GB SSD → OS (`/`)
- 1TB HDD → `/mnt/storage`

### Networking
- ❌ Built-in NIC (enp1s0): 10/100 (fallback only)
- ✅ Active NIC: USB 3.0 Gigabit (`enx6c6e072bdc14`)

### Key Insight
This is **consumer laptop hardware repurposed as a server**:
- perfectly usable
- not redundant
- not designed for 24/7 reliability

---

## 🌐 Network Architecture

### Physical Flow

Internet (Fiber)
→ Fiber Modem / ONT
→ Google WiFi (DMZ target)
→ Switch
→ Server (192.168.86.53)
→ Pi-hole (192.168.86.49)

---

### Logical Responsibilities

#### Fiber Modem
- Provides internet connection
- Forwards all traffic via DMZ to Google WiFi

#### Google WiFi
- Primary router
- Handles:
  - DHCP
  - Routing
  - Port forwarding (80/443)

#### Pi-hole
- DNS server for LAN
- Provides:
  - Ad blocking
  - Split-horizon DNS

---

### Key Concept: Double NAT (Controlled)

There are technically two routers, but:
- DMZ eliminates most problems
- Google WiFi behaves as the “real” router

---

## 🔁 Request Flow (How Traffic Actually Moves)

Client
→ Pi-hole (DNS resolution)
→ Caddy (HTTPS entry point)
→ Internal service (Flask / Jellyfin)

---

## 🔐 Reverse Proxy (Caddy)

### Role
Caddy is the **central entry point** for all web traffic.

It handles:
- HTTPS (Let’s Encrypt certificates)
- Routing to internal services
- Access control

---

### Routing

- `boener.duckdns.org` → Flask (port 5000)
- `jellyboen.duckdns.org` → Jellyfin (port 8096)

---

### Security Model

- TLS terminated at Caddy
- Internal services run on localhost
- Jellyfin restricted to LAN via IP filtering

---

## 🎬 Jellyfin

### Role
Media server for local streaming.

### Access
- URL: https://jellyboen.duckdns.org
- HTTPS handled by Caddy

### Behavior
- Allowed: LAN clients
- Blocked: external internet (403)

### Media
- Stored at: `/mnt/storage`

### Performance
- Hardware transcoding enabled (VAAPI)

---

## 🐍 Flask Application

### Role
Dynamic backend service.

### Details
- Runs on port 5000
- Managed via systemd
- Reverse proxied through Caddy

---

## 💾 Storage

### Mount
- `/mnt/storage` (ext4)

### Access
- Shared via Samba

### Purpose
- Media storage (Jellyfin)
- General file storage

---

## ⚠️ Constraints / Risks

- No backup system (data risk)
- No redundancy (single disk failure = data loss)
- Laptop hardware (thermal + longevity concerns)
- USB NIC dependency

---

## 🧠 System Characterization

This system is best described as:

> A multi-service home server built on consumer hardware,
> using modern web architecture (reverse proxy + HTTPS),
> with strong networking fundamentals but minimal redundancy.

---

## 🧾 Changelog

### 2026-04-19
- Converted document into canonical “living doc” format
- Expanded architecture explanation
- Added network + request flow clarity
- Reflected USB gigabit adapter as primary NIC
- Formalized Caddy as central system component

