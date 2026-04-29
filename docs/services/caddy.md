# 🌐 Caddy Service

## 🧠 Overview

Caddy is the **central web entry point** for the home server.

It is responsible for:
- accepting inbound HTTP/HTTPS traffic
- obtaining and renewing TLS certificates automatically
- reverse proxying requests to internal services
- enforcing some access-control rules

In practical terms, Caddy is the layer that turns a collection of local services into a coherent, domain-based system.

---

## 🏗️ Role in the System

### Request Path

Client
→ DNS resolution (Pi-hole locally, public DNS externally)
→ Caddy
→ internal service

### Why Caddy Matters

Without Caddy:
- services would need to be accessed by raw IP and port
- HTTPS would be harder to manage
- each service would need its own public-facing setup

With Caddy:
- services are accessed by clean hostnames
- TLS is automatic
- routing logic lives in one place

---

## 🌍 Hostnames / Routing

### ianboen.com www.ianboen.com
### boener.duckdns.org (retiring)
Routes to:
- Flask application on `127.0.0.1:5000`

### jellyfin.ianboen.com
### jellyboen.duckdns.org (retiring)
Routes to:
- Jellyfin on `127.0.0.1:8096`

This means Caddy is acting as a **reverse proxy**: it accepts requests at the edge, then forwards them internally.

---

## 🔐 HTTPS / TLS

### Certificate Source
- Let’s Encrypt

### Behavior
Caddy automatically:
- requests certificates
- renews certificates
- serves HTTPS
- redirects HTTP to HTTPS when appropriate

### Key Concept
TLS is tied to the **hostname**, not the raw IP address.

That is why:
- `https://boener.duckdns.org` works cleanly
- `https://192.168.86.53` would not match the certificate

---

## 🧩 Current Configuration Pattern

Current high-level behavior:

- `ianboen.com` `www.ianboen.com` `boener.duckdns.org` → Flask backend
- `jellyfin.ianboen.com` `jellyboen.duckdns.org` → Jellyfin backend
- Jellyfin has an additional LAN-only access rule enforced in Caddy

This makes Caddy both:
- a routing layer
- a light security layer

---

## 🛡️ Access Control

### Jellyfin Rule
Jellyfin via duckdns is intentionally **not public**.

Caddy checks the client IP range and denies non-local traffic from jellyboen.duckdns.com with:
- HTTP 403 Forbidden

### Why This Matters
Even though the domain exists publicly and TLS works publicly, the actual application is blocked unless the request is coming from the local network.

This is a clean pattern because it allows:
- valid public certificates
- simple local access
- no public exposure of the media server itself
- This will change when DuckDNS is retired

---

## 🧠 Relationship to DNS

Caddy depends on DNS working correctly.

### Local Network
- Pi-hole resolves local service names toward the server’s LAN IP

### Public Internet
- Cloudflare resolves ianboen.com toward the WAN IP
- DuckDNS resolves domains toward the WAN IP (retiring)

This creates a **split-horizon DNS** pattern:
- same hostname
- different routing context depending on where the client is

---

## ⚠️ Operational Notes

### Single Point of Entry
If Caddy is down:
- domains stop working
- HTTPS stops working
- reverse proxy access stops working

### Single Point of Policy
Changes to Caddy affect:
- routing
- certificate behavior
- public/private exposure

This makes it powerful, but also means configuration changes should be deliberate.

---

## 🧪 How to Think About Failures

If something behind Caddy stops working, the problem usually falls into one of these buckets:

1. **DNS problem**
   - hostname resolves incorrectly
2. **Caddy problem**
   - bad config, service down, cert issue
3. **Backend problem**
   - Flask or Jellyfin not running
4. **Network policy problem**
   - access blocked intentionally or accidentally

This is a useful mental model for troubleshooting.

---

## 🎯 Summary

Caddy is the server’s **front door**.

It provides:
- hostname-based access
- automatic HTTPS
- routing to internal services
- a simple security boundary for LAN-only apps

For this system, Caddy is one of the most important components because it unifies networking, security, and usability into a single layer.
