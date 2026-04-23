# ☁️ Cloudflare DNS / DDNS

*Last updated: 2026-04-22*

---

## Purpose

This document describes how Cloudflare is used for:

* primary public DNS for `ianboen.com`
* dynamic DNS updates from the home server
* future expansion for subdomains and email-related DNS records

This file should be treated as the canonical reference for Cloudflare-related changes.

---

## Current Role

Cloudflare is the **authoritative DNS provider** for:

* `ianboen.com`
* `www.ianboen.com`

These names currently point to the home server's public WAN IP and terminate at Caddy, which routes requests to Flask.

DuckDNS remains configured separately as a fallback / legacy path, but Cloudflare is now the primary public DNS system.

---

## Current Records in Use

### A Records

* `ianboen.com` → WAN IP (dynamic)
* `www.ianboen.com` → WAN IP (dynamic)

### Proxy Status

* currently set to **DNS only**
* Cloudflare proxy/CDN is **not** currently enabled for these records

Reason:

* keep routing simple during initial domain migration
* allow direct certificate issuance and direct visibility into traffic behavior
* reduce Cloudflare-specific variables during early debugging

---

## Dynamic DNS Design

The server updates Cloudflare DNS records directly through the Cloudflare API.

### Script

* `/home/ian/cloudflare-ddns.sh`

### Config File

* `/home/ian/.cloudflare/ddns.env`

Config contains:

* `CF_TOKEN`
* `CF_ZONE_ID`
* `CF_RECORD_ID_ROOT`
* `CF_RECORD_ID_WWW`

### Permissions

The config file should remain locked down:

```bash
chmod 600 /home/ian/.cloudflare/ddns.env
```

---

## Script Behavior

The updater script currently:

1. loads credentials and record IDs from `~/.cloudflare/ddns.env`
2. fetches the current public IP using:

   * `curl ifconfig.me`
3. updates:

   * `ianboen.com`
   * `www.ianboen.com`
4. leaves Cloudflare proxying disabled (`proxied: false`)

This is a simple record-overwrite model.

It does **not** currently:

* compare old vs new IP before updating
* manage additional subdomains
* manage MX, TXT, SPF, DKIM, or DMARC records
* toggle proxy mode
* update DuckDNS

That simplicity is intentional.

---

## Automation

The updater is scheduled in the user crontab:

```cron
*/5 * * * * /home/ian/cloudflare-ddns.sh >/dev/null 2>&1
```

### Meaning

* runs every 5 minutes
* suppresses normal output
* keeps DNS aligned with WAN IP changes

---

## Relationship to Caddy

Cloudflare handles **public DNS only**.

Caddy handles:

* TLS certificate issuance
* HTTPS termination
* hostname routing to backend services

Current primary route:

* `ianboen.com` → Caddy → Flask
* `www.ianboen.com` → Caddy → Flask

Important distinction:

* Cloudflare decides how the hostname resolves
* Caddy decides what service receives the request

---

## Operational Notes

### Zone ID

Cloudflare operations depend on the zone ID for `ianboen.com`.

### Record IDs

The updater currently uses fixed record IDs for:

* root A record
* `www` A record

This means:

* deleting and recreating records in Cloudflare may break the updater
* if a record is recreated, its new ID must be updated in `ddns.env`

### API Token Scope

The API token should remain narrowly scoped to:

* Zone → DNS → Edit
* specific zone: `ianboen.com`

If exposed:

* revoke it
* create a replacement
* update `ddns.env`

---

## Future Expansion

This document should be updated when any of the following are added or changed:

* new subdomains
* CNAME records
* email forwarding
* MX records
* SPF / DKIM / DMARC
* Cloudflare proxy enablement
* origin hardening rules
* tunnel experiments
* API token design changes
* updater script logic changes

---

## Failure Modes

### 1. Token failure

Symptoms:

* DDNS updates fail
* API returns auth errors

Likely causes:

* revoked token
* expired/rotated token not updated locally
* permission scope too narrow

### 2. Record ID drift

Symptoms:

* script runs but target record is not updated correctly

Likely cause:

* DNS record was deleted/recreated in Cloudflare
* stored record ID is stale

### 3. WAN IP drift

Symptoms:

* domain resolves to old public IP
* local service works, public access fails

Likely causes:

* cron not running
* script failure
* public IP changed recently and update has not occurred yet

### 4. DNS vs app confusion

Symptoms:

* domain resolves correctly but site still fails

Likely causes:

* Caddy misconfiguration
* port forwarding issue
* backend app failure

Cloudflare DNS success does **not** guarantee application success.

---

## Design Philosophy

This setup intentionally keeps the DNS layer simple and explicit:

* Cloudflare is authoritative
* the server updates only the records it owns
* Caddy remains the front door
* application routing remains separate from DNS logic

That separation should be preserved as the system grows.
