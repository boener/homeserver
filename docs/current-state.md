# 🧠 Home Server – Current State

_Last updated: 2026-04-22_

---

## 🌐 Networking

### DHCP Model
- DHCP is provided by the **Google WiFi router**
- The server operates as a **DHCP client**
- A **DHCP reservation** is configured on the router to assign:
  - `192.168.86.53` → server (via USB gigabit adapter MAC)

### Current Behavior
- Server successfully receives `192.168.86.53` via DHCP (dynamic lease)
- Reservation is now functioning correctly
- Default route is via:
  - `192.168.86.1` (router)
  - Interface: `enx6c6e072bdc14`

### Interfaces
- `enx6c6e072bdc14` (USB gigabit adapter)
  - Primary interface
  - Active and in use

- `enp1s0` (onboard ethernet)
  - Configured for DHCP
  - Physically disconnected
  - Retained as **failover / recovery interface**

### Design Decision
The onboard NIC remains configured but unplugged intentionally to allow:
- emergency physical reconnection
- recovery access without needing console/keyboard

This introduces a theoretical dual-DHCP risk if both are connected,
but is considered acceptable given controlled usage.

### Previous Issue (Resolved)
- Router previously issued incorrect IP (`.22`) despite reservation
- Likely caused by:
  - stale DHCP lease
  - interface/MAC mismatch
- Temporary workaround used:
  - static IP assignment on server

### Resolution
- Re-enabled DHCP on server
- Router now correctly honors reservation
- Static configuration removed

---

## 🌍 DNS & Domain (Cloudflare)

### Primary Domain
- `ianboen.com`
- `www.ianboen.com`

### Additional Cloudflare Hostnames
- `jellyfin.ianboen.com`
- `cloud.ianboen.com`

### DNS Provider
- **Cloudflare (primary)**
- DuckDNS retained as **fallback / legacy**

### Dynamic DNS (Cloudflare)

The system uses a custom dynamic DNS updater:

- Script:
  - `/home/ian/cloudflare-ddns.sh`

- Config:
  - `~/.cloudflare/ddns.env`

### Behavior
- Script fetches current public IP via:
  - `curl ifconfig.me`
- Updates relevant Cloudflare records

### Automation
```cron
*/5 * * * * /home/ian/cloudflare-ddns.sh >/dev/null 2>&1
```

---

## 📧 Email (Domain Mail Handling)

### Overview
Custom domain email for `ian@ianboen.com` is implemented using **external providers**, not the home server.

The system provides normal send/receive behavior while avoiding the complexity of self-hosting mail.

---

### Inbound Mail

Handled by **Cloudflare Email Routing**:

```text
Internet sender → Cloudflare → Gmail inbox
```

- Emails sent to `ian@ianboen.com` are forwarded to:
  - `ian.boen@gmail.com`
- No mail services are exposed on the home network

---

### Outbound Mail

Handled via **SMTP2GO** configured inside Gmail:

```text
Gmail → SMTP2GO → recipient
```

- Gmail is configured with a “Send mail as” identity:
  - `ian@ianboen.com`
- SMTP2GO handles actual delivery to the internet

---

### Capability

The address `ian@ianboen.com` now:

- sends and receives email normally through Gmail
- behaves as a standard mailbox from the user perspective

---

### DNS Impact

Cloudflare DNS now includes additional records required for SMTP2GO setup:

- CNAME records for provider verification and/or DKIM

Exact values are managed directly in Cloudflare.

---

### Design Decision

This approach was chosen to avoid:

- running a local SMTP server
- exposing mail ports
- dealing with IP reputation and spam filtering from a residential IP

The system intentionally keeps email **out of the home server’s responsibility**.

---

### Account / Credential Notes

- SMTP2GO account is tied to an external account (non-personal email used during setup)
- Credentials are stored in Gmail configuration
- Credentials should be recoverable via the provider account if needed

---

### Risks / Future Work

- free-tier limits may change
- deliverability may require:
  - SPF
  - DKIM
  - DMARC tuning

---

## 🌐 Web Entry / Routing

### Caddy Public Routes
- `ianboen.com`, `www.ianboen.com` → Flask (`127.0.0.1:5000`)
- `jellyfin.ianboen.com` → Jellyfin (`127.0.0.1:8096`)

---

## 💽 Backup System

### Overview
The system uses a **two-layer backup architecture** with explicit directory separation:

- `/mnt/backup/storage` → data backups (user)
- `/mnt/backup/system` → system backups (root)

---

## 🧾 Changelog

### 2026-04-20
- Refined backup structure to separate `/mnt/backup/storage` and `/mnt/backup/system`
