# 🧾 Changelog

## 2026-04-24 — Cowrie Honeypot Behavior Refinement

### Changed
- Introduced active `cowrie.cfg` override (previously using defaults)
- Set custom honeypot hostname to `web-prod-nyc01`
- Enabled explicit `UserDB` authentication mode
- Created and activated `userdb.txt`

### Improved
- Authentication realism:
  - Common weak credentials (`root/root`, `root/123456`, etc.) now fail
  - Most other credentials succeed to encourage deeper attacker interaction
- Honeypot now behaves more like a "slightly misconfigured real server" instead of a default demo environment

### Insight
- Learned that SSH client default key-based auth can interfere with testing Cowrie password behavior
- Verified correct testing requires forcing password authentication

---

## 2026-04-24 — Cowrie SSH Honeypot Deployed

### Added
- Deployed Cowrie SSH honeypot in Docker
- Exposed honeypot on host port 2222 (container 2222 → host 2222) (safe non-standard port)
- Implemented persistent logging to host filesystem
- Capturing:
  - login attempts (username/password)
  - interactive shell commands
  - session metadata
- Added persistent storage for:
  - logs (`~/cowrie/log`)
  - downloads (`~/cowrie/downloads`)
  - TTY session recordings (`~/cowrie/tty`)

### Fixed
- Resolved container restart loop caused by incorrect bind mounts overriding Cowrie internal filesystem
- Resolved permission errors preventing log and session file creation

### Verified
- Local SSH access to honeypot
- LAN access from external devices
- Full interactive fake shell behavior
- Command logging in `cowrie.json`

### Notes
- Real SSH service remains untouched and separate
- Honeypot is not yet exposed to the internet (no router port forwarding configured)
- Future work: expose safely, build live UI from JSON logs

## 2026-04-22
- Added SMTP2GO for outbound email via Gmail
- Added CNAME records in Cloudflare for SMTP2GO verification
- Configured Gmail to send mail as `ian@ianboen.com` via SMTP2GO

## 2026-04-22
- Added Cloudflare subdomains `jellyfin.ianboen.com` and `cloud.ianboen.com`
- Extended Cloudflare DDNS script to manage additional DNS records
- Integrated new records into secure config (`ddns.env`)
- Updated Caddy routing to:
  - expose Jellyfin publicly via `jellyfin.ianboen.com`
  - maintain LAN-restricted access via `jellyboen.duckdns.org`
  - add placeholder route for `cloud.ianboen.com`
- Implemented split-horizon DNS in Pi-hole for all primary hostnames
- Enabled Cloudflare proxy (orange cloud) for `jellyfin.ianboen.com`
- Configured Cloudflare SSL mode to **Full (strict)**
- Updated DDNS script to preserve proxy mode for Jellyfin record
- Verified full public path: Client → Cloudflare → Router → Caddy → Jellyfin
- Maintained DNS-only mode for primary domain to preserve observability during development

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
