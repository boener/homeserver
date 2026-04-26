# 🧾 Changelog

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
