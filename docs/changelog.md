# 🧾 Changelog

## 2026-04-29 — WireGuard VPN (MacBook Node)

### Added
- WireGuard VPN service on secondary node (`macbook`)
- New service documentation: `docs/services/wireguard-vpn.md`

### Configuration
- VPN subnet: `10.0.0.0/24`
- Server address: `10.0.0.1`
- Client addressing model: static (`10.0.0.x`)
- Public endpoint: `ianboen.com:51820`
- Router forward: UDP `51820` → `192.168.86.45:51820`
- DNS for clients: Pi-hole (`192.168.86.49`)

### Behavior
- Full-tunnel VPN (all client traffic routed through home network)
- NAT via `iptables MASQUERADE` on `macbook`
- IP forwarding enabled on interface startup
- Service managed via `wg-quick@wg0`
- Enabled at boot and verified across reboot

### Impact
- Introduces a trusted remote access path into the LAN
- Does not affect existing services (Cowrie remains on TCP `22`)

### Notes
- Key material currently stored in `/home/ian`
- Planned future move to `/etc/wireguard` for tighter control

## 2026-04-26 — Outbound Email (SMTP2Go) Integration

### Added
- Outbound email capability via Postfix + SMTP2Go
- New service documentation: `docs/services/outbound-email.md`

### Changed
- Primary server now includes outbound email as a core service

### Fixed
- Resolved SMTP authentication issues caused by relayhost mismatch
- Resolved sender rejection due to unverified domain (`ian@ubuntu`)

### Configuration
- Configured Postfix as local MTA
- Configured authenticated SMTP relay via SMTP2Go (`[mail.smtp2go.com]:587`)
- Enabled TLS encryption for outbound mail
- Implemented sender canonical rewrite to `ian@ianboen.com`

### Insight
- SMTP relay requires exact host matching between relayhost and SASL map
- External mail providers enforce verified sender domains
- Local MTA pattern (Postfix + mailutils) creates a clean abstraction for alerting

---

## 2026-04-26 — Backup System Hardening

### Changed
- Updated backup drive mount to use `nofail` and `x-systemd.automount`
- Improved reliability of external USB backup drive mounting

### Added
- Explicit mount safety checks to:
  - `/home/ian/backup.sh`
  - `/usr/local/sbin/system-backup.sh`

### Fixed
- Prevented backups from writing to root filesystem when `/mnt/backup` is not mounted
- Corrected storage backup directory layout (`/mnt/backup/storage/...`)

### Documentation
- Added `docs/services/backup.md` with:
  - script locations
  - cron schedules
  - mount behavior
  - directory structure

### Insight
- Backup systems must fail safely, not silently
- Mount-dependent scripts must explicitly verify mount state before writing

---

## 2026-04-25 — Added MacBook Node

### Added
- New secondary node: `macbook`
- Created `docs/nodes/macbook.md`

### Changed
- Refactored system documentation from single-node → multi-node model
- Updated `current-state.md` to explicitly define nodes
- Updated `architecture.md` to include node layer and future distribution model

### Insight
- System is transitioning from a single host into a flexible multi-node lab environment
- Clear separation between "primary server" and "auxiliary nodes" improves reasoning and future scaling

---

## 2026-04-25 — Unified Lid-Aware Backlight Control

### Changed
- Standardized backlight control across all lab nodes

---

## 2026-04-24 — Cowrie Honeypot Behavior Refinement

(unchanged)
