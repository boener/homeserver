# 🧾 Changelog

## 2026-04-22
- Migrated primary DNS from DuckDNS to Cloudflare
- Registered and configured `ianboen.com` domain
- Implemented custom Cloudflare dynamic DNS updater (`cloudflare-ddns.sh`)
- Added secure credential storage (`~/.cloudflare/ddns.env`) with restricted permissions
- Scheduled automated DDNS updates via cron (every 5 minutes)
- Updated Caddy configuration to serve `ianboen.com` and `www.ianboen.com`
- Successfully issued Let's Encrypt certificates for new domain via Caddy
- Verified full public request path: Internet → Cloudflare → Router → Caddy → Flask
- Updated architecture to reflect Cloudflare as primary DNS provider
- Retained DuckDNS as fallback / legacy access path
- Added `docs/services/cloudflare-ddns.md` for ongoing DNS management and future expansion

## 2026-04-22
- Restored DHCP-based networking on server (removed static IP workaround)
- Confirmed router DHCP reservation correctly assigns `192.168.86.53`
- Documented previous DHCP lease conflict issue and resolution
- Clarified dual-interface strategy (USB primary + onboard fallback)
- Updated `current-state.md` with full networking model and design rationale

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
