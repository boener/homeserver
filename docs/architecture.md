# 🏗️ System Architecture

## 🧠 Overview

This document describes the **structural design** of the home server.

`current-state.md` answers:
- what exists right now
- what IPs, services, and paths are in use

This document answers:
- how the system is organized
- where responsibilities are divided
- how requests move through the system
- where failures are likely to occur
- why the system is shaped this way

---

## 🧱 Layered Architecture

The system is easiest to understand as a stack of layers.
Each layer has a distinct role.

### 1. Edge / Internet Layer
This is the public-facing edge of the system.

Components:
- Cloudflare DNS
- Cloudflare-managed domain (`ianboen.com`)
- Cloudflare Email Routing
- DuckDNS (fallback / legacy)
- Public DNS resolution
- Inbound internet traffic

Role:
- gives the system stable public names even though the WAN IP may change
- allows public clients to find the home network
- separates authoritative DNS management from the server itself
- keeps email handling external to the home server

Current model:
- `ianboen.com` and `www.ianboen.com` are the **primary public hostnames** for the Flask app
- `jellyfin.ianboen.com` is a Cloudflare-managed public hostname for Jellyfin
- `cloud.ianboen.com` exists as a Cloudflare-managed placeholder hostname and currently serves a stub response
- Cloudflare is the authoritative DNS provider for the primary domain
- the server updates Cloudflare A records via API using a custom DDNS script
- DuckDNS remains available as a fallback / legacy access path

Key idea:
The internet does **not** connect directly to Flask or Jellyfin.
It connects to a hostname, which resolves through Cloudflare (or DuckDNS for fallback cases), and eventually routes toward the home network.

---

### 2. Network Layer
This is the routing and packet-delivery layer inside the home setup.

Components:
- Fiber modem / ONT
- Google WiFi router
- LAN subnet (`192.168.86.x`)
- Switch

Role:
- provides NAT, routing, DHCP, and port forwarding
- delivers public inbound traffic to the correct internal host
- connects internal devices to each other

Current model:
- modem forwards traffic via DMZ to Google WiFi
- Google WiFi behaves as the primary practical router
- Google WiFi forwards ports `80` and `443` to the server

---

### 3. DNS Layer
This is the naming and resolution layer.

Components:
- Pi-hole
- local DNS overrides
- Cloudflare public DNS (primary)
- DuckDNS (fallback records)

Role:
- resolves names for LAN clients
- provides ad blocking
- creates split-horizon behavior

Current pattern:
- inside the LAN, primary hostnames (`ianboen.com`, `www.ianboen.com`, `jellyfin.ianboen.com`, `cloud.ianboen.com`) resolve to the server's LAN IP (`192.168.86.53`)
- fallback hostnames (`*.duckdns.org`) also resolve locally via overrides
- outside the LAN, primary hostnames resolve via Cloudflare to the WAN IP
- DuckDNS remains available as a secondary resolution path

Key idea:
The same hostname can lead to different network paths depending on where the client is located.

Reference:
- See `docs/services/cloudflare-ddns.md` for implementation details of Cloudflare DNS and dynamic updates

---

### 4. Entry Layer
This is the single web entry point into the server.

Component:
- Caddy

Role:
- accepts incoming HTTP / HTTPS traffic
- terminates TLS
- selects the correct backend based on hostname
- enforces some access-control rules
- serves placeholder responses for unfinished hostnames

Current routing:
- `ianboen.com`, `www.ianboen.com` → Flask backend (primary)
- `jellyfin.ianboen.com` → Jellyfin backend (public)
- `cloud.ianboen.com` → stub / placeholder response (`not configured yet`)
- `boener.duckdns.org` → Flask backend (fallback)
- `jellyboen.duckdns.org` → Jellyfin backend (LAN-restricted fallback)

Key idea:
Caddy is the front door.

---

### 5. Application Layer
Components:
- Flask (public app)
- Jellyfin (public via `jellyfin.ianboen.com`, LAN-restricted via DuckDNS fallback)
- placeholder web response for `cloud.ianboen.com`

Notes:
- the primary website and Jellyfin are both reverse-proxied through Caddy
- the `cloud.ianboen.com` hostname currently exists only as a reserved entry point and does not yet map to a real application

---

### 6. Email Handling Layer (External to Server)
Email for the domain is intentionally **not** hosted on the home server.

Components:
- Cloudflare Email Routing (inbound)
- Gmail mailbox / identity configuration
- SMTP2GO (outbound delivery)

Role:
- receives mail for `ian@ianboen.com`
- lets Gmail send mail as the custom domain
- avoids exposing SMTP/IMAP services from the house

Current model:
- inbound mail path: `Internet sender → Cloudflare Email Routing → Gmail inbox`
- outbound mail path: `Gmail → SMTP2GO → recipient`
- Cloudflare DNS contains the additional records needed for SMTP2GO verification and/or DKIM

Key idea:
Email is part of the domain architecture, but **not part of the home server runtime**.

---

## 🔁 Request Lifecycle

### A. Public Request Path (Primary Website)
Example: `https://ianboen.com`

1. DNS resolves via Cloudflare
2. Request hits WAN IP
3. Router forwards to server
4. Caddy terminates TLS
5. Request proxied to Flask

---

### B. Public Request Path (Jellyfin)
Example: `https://jellyfin.ianboen.com`

1. DNS resolves via Cloudflare
2. Request hits WAN IP
3. Router forwards to server
4. Caddy terminates TLS
5. Request proxied to Jellyfin

---

### C. Local Request Path
Example: LAN client → `https://ianboen.com`

1. Pi-hole resolves to LAN IP
2. Client connects directly to Caddy
3. Caddy routes to backend

---

### D. Placeholder Host Path
Example: `https://cloud.ianboen.com`

1. DNS resolves via Cloudflare
2. Request hits WAN IP
3. Router forwards to server
4. Caddy terminates TLS
5. Caddy returns a stub response indicating the service is not configured yet

---

### E. Fallback Path (DuckDNS)
Example: `https://boener.duckdns.org`

- Same path, but uses DuckDNS for resolution

---

### F. Email Flow (Separate from Web Traffic)
Inbound example: mail to `ian@ianboen.com`

1. Sender targets domain MX/routing managed externally
2. Cloudflare Email Routing forwards the message
3. Message arrives in Gmail inbox

Outbound example: mail sent from Gmail as `ian@ianboen.com`

1. User sends from Gmail
2. Gmail authenticates to SMTP2GO
3. SMTP2GO delivers to recipient

---

## 🎯 Mental Model Summary

- DNS decides **where the client goes**
- Caddy decides **what service gets the request**
- the app decides **how it behaves**
- email for the domain is **externalized** and does not run on the server
