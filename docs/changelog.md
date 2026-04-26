# 🧾 Changelog

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
