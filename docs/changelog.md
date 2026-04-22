# 🧾 Changelog

## 2026-04-20
- Refined backup directory structure (`/mnt/backup/storage` vs `/mnt/backup/system`)
- Updated storage backup script to use new path
- Improved safety and isolation between backup types
- Expanded backup system to dual-layer model (data + system)
- Added root-level system backup script for `/etc` and `/home/ian`
- Introduced privilege-separated backup architecture
- Added staggered cron scheduling (3:00 data, 3:30 system)
- Improved recovery model (file-level restore, system rebuild, full data restore)
- Added local snapshot backup system and external backup disk
- Implemented daily automated backups and retention policy
- Changed system timezone to America/Phoenix

## 2026-04-19
- Initialized structured documentation system
- Created current-state.md
- Added architecture documentation
