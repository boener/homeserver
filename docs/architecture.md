# 🏗️ System Architecture

## 🧠 Overview

This system now consists of **multiple nodes**, not a single server.

---

## 🖥️ Node Layer (New)

### Primary Node: `ubuntu`
- Hosts all production-facing services
- Acts as the main application server

### Secondary Node: `macbook`
- Lab / auxiliary node
- Used for experimentation, testing, and future service distribution

Key idea:
The system is evolving from **single-node → multi-node architecture**.

---

## 🧱 Layered Architecture

### 1. Edge / Internet Layer
- Cloudflare DNS
- Public domain (`ianboen.com`)

### 2. Network Layer
- Google WiFi router
- LAN: `192.168.86.x`

### 3. DNS Layer
- Pi-hole
- Cloudflare

### 4. Entry Layer (Web)
- Caddy (runs on primary node)

### 5. Parallel Ingress (Honeypot)
- Cowrie (primary node, Docker)

### 6. Application Layer
- Flask (primary)
- Jellyfin (primary)

### 7. Expansion Path

Future direction:

```text
Multiple nodes → distributed services
```

Possible evolution:
- move services to separate machines
- isolate workloads (media, honeypot, web)

---

## 🎯 Key Shift

Previously:

```text
Internet → single server → everything
```

Now:

```text
Internet → router → primary node → services
                     ↘ secondary nodes (lab / future roles)
```
