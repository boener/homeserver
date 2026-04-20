# 🧠 Home Server – Current State

_Last updated: 2026-04-20_

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

## 🧭 Operational Posture

This machine is best understood as:
- an always-on learning and experimentation server
- low-risk and non-critical to daily life
- remotely administered most of the time
- acceptable to lose and rebuild if necessary

This is **not** treated as production infrastructure.
It is closer to a persistent lab environment or a recoverable game save than a mission-critical system.

---

## 🔐 Access / Administration Model

### Human Access
- Primary administration is over SSH from Ian's Windows 11 machine
- Secondary SSH access is occasionally used from an iMac or Chromebook
- Physical access is possible, but inconvenient because the machine is not kept in an easy-to-reach location

### Admin User Model
- Primary human operator: `ian`
- Administrative escalation is done with `sudo`
- Direct root login is not part of the normal workflow

### SSH Model
- SSH runs on the default port: `22`
- SSH keys are configured on the Windows administration machine
- Password authentication remains enabled as a fallback

### Why Password SSH Is Still Enabled
This is a deliberate tradeoff, not an accident.
Because the project is a learning system and network settings have already changed during experimentation, password login remains available to reduce the risk of accidental lockout.

---

## 🌐 Network Architecture

### Physical Flow

Internet (Fiber)
→ Fiber Modem / ONT
→ Google WiFi (DMZ target)
→ Switch
→ Server (`192.168.86.53`)
→ Pi-hole (`192.168.86.49`)

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

### Server IP Model
- Current server address: `192.168.86.53`
- At the moment, this IP is manually configured in Ubuntu
- This is a temporary state created during the network adapter transition to the USB gigabit NIC
- The long-term intent is to return to DHCP with a reservation in Google WiFi once lease / MAC confusion settles down

### Important Note
This means addressing is currently under **OS-level manual control**, not purely router-driven DHCP behavior.

---

### DNS Behavior
- Google WiFi uses Pi-hole as its DNS resolver
- Most client DNS traffic reaches Pi-hole through the router
- Ian's Windows PC is pointed directly at Pi-hole for clearer per-device visibility

### Visibility Implication
Pi-hole currently sees:
- the router as a bulk client for most devices
- Ian's Windows machine as a distinct client

So DNS visibility is **partially aggregated**, not fully per-device across the whole network.

---

### Local DNS / Split-Horizon Entries
The following public hostnames are overridden internally to the server's LAN IP:

- `boener.duckdns.org` → `192.168.86.53`
- `jellyboen.duckdns.org` → `192.168.86.53`

This creates a split-horizon DNS pattern:
- inside the LAN, these names resolve directly to the server
- outside the LAN, they resolve via DuckDNS to the WAN IP

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

### Verified Runtime Facts
- Config path: `/etc/caddy/Caddyfile`
- systemd unit: `/usr/lib/systemd/system/caddy.service`
- Binary: `/usr/bin/caddy`
- Service name: `caddy.service`
- Runtime user/group: `caddy:caddy`
- Service is enabled and running

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
- URL: `https://jellyboen.duckdns.org`
- HTTPS handled by Caddy

### Behavior
- Allowed: LAN clients
- Blocked: external internet (403 via Caddy)

### Media
- Stored at: `/mnt/storage`

### Performance
- Hardware transcoding enabled (VAAPI)

### Verified Runtime Facts
- systemd unit: `/usr/lib/systemd/system/jellyfin.service`
- systemd override directory present: `/etc/systemd/system/jellyfin.service.d`
- Observed override file: `jellyfin.service.conf`
- Binary: `/usr/bin/jellyfin`
- ffmpeg binary: `/usr/lib/jellyfin-ffmpeg/ffmpeg`
- Runtime user/group: `jellyfin:jellyfin`
- Service is enabled and running

---

## 🐍 Flask Application

### Role
Dynamic backend / experimentation service.

### Details
- Runs on port 5000
- Reverse proxied through Caddy
- Managed via systemd

### Verified Runtime Facts
- Application path: `/home/ian/flaskapp/`
- Virtual environment present in app directory
- systemd unit: `/etc/systemd/system/flaskapp.service`
- Launch command: `/home/ian/flaskapp/bin/python /home/ian/flaskapp/app.py`
- Runtime user/group: `ian:ian`
- Service is enabled and running

---

## 💾 Storage

### Mount
- `/mnt/storage` (ext4)

### Access
- Shared via Samba

### Purpose
- Media storage (Jellyfin)
- General file storage

### Samba Shares
- `[storage]` → `/mnt/storage` (read/write for `ian`)
- `[home]` → `/home/ian` (read/write for `ian`)
- `[root_read_only]` → `/` (read-only for `ian`)

### Notes
This is intentionally a convenience-heavy administration model.
It makes remote browsing and troubleshooting easier, but also exposes broad filesystem visibility over Samba to the authenticated admin user.

---

## 🌍 External Exposure Model

### Publicly Reachable
- `boener.duckdns.org` → publicly accessible through Caddy and Flask

### Public DNS but Intentionally Blocked
- `jellyboen.duckdns.org` → publicly resolvable, but external clients are denied by Caddy

This is an intentional pattern:
- valid public hostnames and certificates still exist
- local access remains simple
- Jellyfin is not actually exposed to the internet

---

## 🔁 Background Automation

### DuckDNS
- Dynamic DNS updates are handled by a script run from cron
- Update frequency: every 5 minutes

This keeps public DNS aligned with the current WAN IP.

---

## ⚠️ Constraints / Risks

- No backup system (data risk)
- No redundancy (single disk failure = data loss)
- Laptop hardware (thermal + longevity concerns)
- USB NIC dependency
- Password SSH remains enabled for recovery convenience
- Network addressing is temporarily in a transitional state during the NIC / DHCP reservation cleanup

---

## 🧠 System Characterization

This system is best described as:

> A multi-service home server built on consumer hardware,
> using modern web architecture (reverse proxy + HTTPS),
> with strong networking fundamentals, remote-first administration,
> and intentionally low production expectations.

---

## 🧾 Changelog

### 2026-04-20
- Added operational posture and access model
- Documented SSH fallback strategy and admin workflow
- Clarified current static-IP transition state after NIC change
- Documented Pi-hole / Google WiFi DNS visibility behavior
- Added verified runtime facts for Caddy, Jellyfin, and Flask
- Added Samba share definitions
- Added external exposure model and DuckDNS automation note

### 2026-04-19
- Converted document into canonical “living doc” format
- Expanded architecture explanation
- Added network + request flow clarity
- Reflected USB gigabit adapter as primary NIC
- Formalized Caddy as central system component
