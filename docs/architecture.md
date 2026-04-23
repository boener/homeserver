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

Key idea:
There are technically multiple routing devices, but the DMZ setup reduces most of the usual double-NAT problems.

---

### 3. DNS Layer
This is the naming and resolution layer.

Components:
- Pi-hole
- local DNS overrides
- DuckDNS public records

Role:
- resolves names for LAN clients
- provides ad blocking
- creates split-horizon behavior

Current pattern:
- inside the LAN, public hostnames such as `boener.duckdns.org` and `jellyboen.duckdns.org` resolve to the server's LAN IP (`192.168.86.53`)
- outside the LAN, the same hostnames resolve to the WAN IP via DuckDNS

Key idea:
The same hostname can lead to different network paths depending on where the client is located.
That is the core of the split-horizon DNS design.

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
- `boener.duckdns.org` → Flask backend
- `jellyboen.duckdns.org` → Jellyfin backend

Key idea:
Caddy is the front door.
Backends stay internal while Caddy handles certificates, routing, and the first layer of policy.

---

### 5. Application Layer
This is where service-specific behavior lives.

Components:
- Flask
- Jellyfin

Role:
- Flask serves public experimentation / toy web applications
- Jellyfin serves LAN media streaming

Architecture pattern:
- applications do not need to manage public TLS themselves
- applications do not need their own direct internet-facing network setup
- Caddy handles edge concerns, while the apps handle behavior

Key idea:
This keeps responsibilities separated:
- Caddy = networking / certificates / policy
- apps = content / logic / media behavior

---

### 6. Data Layer
This is where persistent files live and how they are moved around.

Components:
- `/mnt/storage`
- `/mnt/storage/media`
- Samba shares

Role:
- stores media and general files
- provides convenient file movement from other machines
- supports Jellyfin's media access

Current access pattern:
- Ian copies files in over Samba
- media files land in storage
- group-based access through the `media` group allows Jellyfin to read them

Key idea:
The data layer is intentionally convenience-heavy, with permissive top-level storage and a more structured media subgroup model inside it.

---

## 🔁 Request Lifecycle

The system behaves differently depending on where the request originates.

### A. Public Request Path (Flask)
Example: a public browser visiting `https://boener.duckdns.org`

1. Client resolves hostname through public DNS / DuckDNS
2. Request reaches WAN IP
3. Modem forwards toward Google WiFi via DMZ
4. Google WiFi forwards ports `80/443` to the server
5. Caddy receives the request
6. Caddy terminates TLS
7. Caddy matches hostname to Flask backend
8. Request is proxied to Flask on localhost
9. Flask returns response through Caddy back to client

---

### B. Local Request Path (Flask or Jellyfin)
Example: LAN client visiting `https://jellyboen.duckdns.org`

1. Client asks Pi-hole to resolve hostname
2. Pi-hole returns the server's LAN IP (`192.168.86.53`)
3. Client connects directly to Caddy on the LAN
4. Caddy terminates TLS
5. Caddy routes request by hostname
6. Backend responds through Caddy back to client

Key difference:
Local clients do **not** need to hairpin through the WAN path when split-horizon DNS is working correctly.

---

### C. Public Request Path to Jellyfin
Example: external client visiting `https://jellyboen.duckdns.org`

1. Public DNS resolves hostname to WAN IP
2. Request reaches Caddy through the forwarded web ports
3. Caddy matches the Jellyfin hostname
4. Caddy checks the client IP policy
5. Non-LAN client is denied with `403`

Key idea:
Jellyfin has a public hostname and valid certificates, but not public application access.
That is an intentional separation between **DNS reachability** and **service exposure**.

---

## 🛡️ Trust Boundaries

The system has several important boundaries.

### Boundary 1: Internet ↔ Home Network
Crossed when public traffic enters through the WAN-facing path.

Controlled by:
- modem / DMZ behavior
- Google WiFi port forwarding
- Caddy as the receiving service

Risk:
Any mistake here can accidentally expose more of the system than intended.

---

### Boundary 2: Edge Proxy ↔ Internal Services
Crossed when Caddy forwards requests to Flask or Jellyfin.

Controlled by:
- Caddy hostname routing
- localhost backend design
- Caddy access rules

Risk:
Bad proxy configuration can misroute traffic or weaken exposure boundaries.

---

### Boundary 3: Human User ↔ Service Users
Crossed when files or permissions need to work for both Ian and service accounts.

Controlled by:
- Unix users and groups
- shared `media` group
- Samba access model
- filesystem permissions

Risk:
This is the main place where “it exists, but the service can't read it” class bugs come from.

---

### Boundary 4: LAN ↔ Public Access Policy
Especially important for Jellyfin.

Controlled by:
- Caddy IP-based filtering
- DNS behavior

Risk:
A service can appear public at the DNS level while still being intended as private.
If the access rule is removed or weakened, exposure changes immediately.

---

## 💥 Failure Domains

The system is small, but failures still cluster into predictable buckets.

### 1. DNS Failure Domain
Symptoms:
- hostname doesn't resolve
- local access fails but direct IP works
- split-horizon behavior breaks

Likely components:
- Pi-hole
- local DNS override entries
- DuckDNS

---

### 2. Network / Routing Failure Domain
Symptoms:
- service works locally but not externally
- port forwarding stops working
- WAN path breaks while LAN path still works

Likely components:
- modem / DMZ setup
- Google WiFi forwarding
- IP assignment problems

---

### 3. Edge / Proxy Failure Domain
Symptoms:
- hostnames resolve but site doesn't load correctly
- certificate issues
- wrong backend content served
- public/private exposure rules behave incorrectly

Likely component:
- Caddy

---

### 4. Application Failure Domain
Symptoms:
- Caddy is healthy but backend returns errors
- Flask app crashes
- Jellyfin UI loads poorly or media behavior fails

Likely components:
- Flask service
- Jellyfin service
- application-specific config or dependencies

---

### 5. Data / Permission Failure Domain
Symptoms:
- files exist but Jellyfin cannot read them
- Samba copies succeed but app behavior breaks
- inconsistent ownership / group access

Likely components:
- `/mnt/storage`
- `media` group model
- Samba write path
- filesystem permissions

---

## 🎯 Design Principles

The current architecture reflects several intentional priorities.

### 1. Simplicity over purity
This system is designed to be understandable and recoverable, not maximally elegant.

Examples:
- password SSH fallback remains enabled
- top-level storage permissions are permissive
- a single reverse proxy handles all web entry

---

### 2. One front door
Caddy is the main web entry point.
This keeps certificates, hostname routing, and exposure policy centralized.

---

### 3. Public names, selective exposure
A hostname being public does **not** automatically mean the backend is public.
That distinction is a core design choice.

---

### 4. Remote-first administration
The machine is operated mostly over SSH and Samba, not by sitting in front of it.
That affects how management, recovery, and convenience decisions are made.

---

### 5. Learning value matters
This is not only a utility server.
It is also a teaching surface.
Some choices are intentionally pragmatic so experimentation stays easy.

---

## 🧠 Mental Model Summary

The simplest accurate way to think about this system is:

> A small layered home server where DNS points clients to Caddy,
> Caddy acts as the web front door,
> applications remain internal,
> and storage is shared through a convenience-first permission model.

More bluntly:
- DNS decides **where the client goes**
- Caddy decides **what service gets the request**
- the app decides **how it behaves**
- permissions decide **whether the data is usable**

That is the architecture.
