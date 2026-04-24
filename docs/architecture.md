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

Key idea:
The internet connects to **multiple ingress paths**, not just the web stack.

---

### 2. Network Layer
This is the routing and packet-delivery layer inside the home setup.

Components:
- Fiber modem / ONT
- Google WiFi router
- LAN subnet (`192.168.86.x`)

Role:
- provides NAT, routing, DHCP, and port forwarding
- delivers public inbound traffic to the correct internal host

Key idea:
The router determines which *type of traffic* goes to which internal service.

---

### 3. DNS Layer
This is the naming and resolution layer.

Components:
- Pi-hole
- local DNS overrides
- Cloudflare public DNS
- DuckDNS (fallback)

Role:
- resolves names for LAN clients
- enables split-horizon DNS

---

### 4. Entry Layer (Web Stack)

Component:
- Caddy

Role:
- handles HTTP / HTTPS only
- terminates TLS
- routes based on hostname

Key idea:
Caddy is the **web entry point**, not the only entry point into the system.

---

### 5. Parallel Ingress: Honeypot Layer

Component:
- Cowrie (Docker container)

Role:
- receives SSH traffic intended for attackers/bots
- simulates a vulnerable system
- captures behavior for analysis and display

Architecture:

```text
Internet TCP/22
    → Router port forward
    → 192.168.86.53:2222
    → Cowrie container (2222)
```

Key idea:
- This path **bypasses Caddy entirely**
- It is intentionally exposed to hostile traffic
- It is not part of the normal application stack

---

### 6. Application Layer

Components:
- Flask (public app)
- Jellyfin

Additional role:
- Flask will act as a **presentation layer** for Cowrie data

Data flow:

```text
Cowrie → JSON logs → Flask → Website UI
```

Key idea:
Flask is both:
- a normal web app
- a consumer of honeypot data

---

### 7. Email Handling Layer (External)

Components:
- Cloudflare Email Routing
- Gmail
- SMTP2GO

Role:
- handles all email outside the home server

---

## 🔁 Request / Data Flows

### A. Web Traffic (Normal Users)

```text
Internet → Cloudflare → Router 80/443 → Caddy → Flask/Jellyfin
```

---

### B. Honeypot Traffic (Attackers)

```text
Internet → Router 22 → Cowrie
```

- no TLS
- no Caddy
- no authentication barrier
- intentionally exposed

---

### C. Honeypot Data Pipeline

```text
Attacker → Cowrie → cowrie.json → Flask → Browser UI
```

---

### D. Local Traffic

```text
LAN client → Pi-hole → Caddy → apps
```

---

### E. Email Flow

```text
Internet → Cloudflare Email Routing → Gmail
Gmail → SMTP2GO → recipient
```

---

## 🎯 Mental Model Summary

- The system has **multiple ingress paths**:
  - Web (Caddy)
  - Honeypot (Cowrie)

- Caddy handles **trusted user traffic**
- Cowrie handles **untrusted hostile traffic**

- Flask becomes a **bridge**:
  - serving users
  - visualizing attacker behavior

- Email remains **external to the system runtime**
