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
- DuckDNS (fallback / legacy)
- Public DNS resolution
- Inbound internet traffic

Role:
- gives the system stable public names even though the WAN IP may change
- allows public clients to find the home network
- separates authoritative DNS management from the server itself

Current model:
- `ianboen.com` and `www.ianboen.com` are the **primary public hostnames**
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
- inside the LAN, primary hostnames (`ianboen.com`, `www.ianboen.com`) resolve to the server's LAN IP (`192.168.86.53`)
- fallback hostnames (`*.duckdns.org`) also resolve locally via overrides
- outside the LAN, primary hostnames resolve via Cloudflare to the WAN IP
- DuckDNS remains available as a secondary resolution path

Key idea:
The same hostname can lead to different network paths depending on where the client is located.

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

Current routing:
- `ianboen.com`, `www.ianboen.com` → Flask backend (primary)
- `boener.duckdns.org` → Flask backend (fallback)
- `jellyboen.duckdns.org` → Jellyfin backend (LAN-restricted)

Key idea:
Caddy is the front door.

---

### 5. Application Layer
- Flask (public app)
- Jellyfin (LAN media server)

---

## 🔁 Request Lifecycle

### A. Public Request Path (Primary Domain)
Example: `https://ianboen.com`

1. DNS resolves via Cloudflare
2. Request hits WAN IP
3. Router forwards to server
4. Caddy terminates TLS
5. Request proxied to Flask

---

### B. Local Request Path
Example: LAN client → `https://ianboen.com`

1. Pi-hole resolves to LAN IP
2. Client connects directly to Caddy
3. Caddy routes to backend

---

### C. Fallback Path (DuckDNS)
Example: `https://boener.duckdns.org`

- Same path, but uses DuckDNS for resolution

---

## 🎯 Mental Model Summary

- DNS decides **where the client goes**
- Caddy decides **what service gets the request**
- the app decides **how it behaves**
