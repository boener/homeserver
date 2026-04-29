# MacBook Node

_Last updated: 2026-04-25_

## Role

Secondary Ubuntu lab node.

This machine serves as the VPN ingress point for the home network while also functioning as a general-purpose lab node.

## System Identity

- Hostname: `macbook`
- User: `ian`
- OS: Ubuntu 24.04 Server
- LAN IP: `192.168.86.45`
- Addressing model: DHCP reservation on the Google WiFi router
- Primary network interface: `enp2s0f0`

## Services

- WireGuard VPN → `docs/services/wireguard-vpn.md`
- Backlight control → `docs/services/backlight-control.md`

## Hardware Summary

- Apple MacBook Pro 8,1
- CPU: Intel Core i5-2415M @ 2.30 GHz
- Architecture: 64-bit x86
- Memory: 4 GiB DDR3
- Storage: 256 GB SATA SSD
- Ethernet: Broadcom NetXtreme BCM57765 Gigabit Ethernet
- Wi-Fi: Broadcom BCM4331 802.11a/b/g/n
- Display/backlight exposed through `/sys/class/backlight/acpi_video0/`

## Networking

The MacBook node is connected by wired Ethernet through interface `enp2s0f0`.

Current expected LAN identity:

```text
macbook → 192.168.86.45
```

The IP address is reserved on the router rather than statically configured on the host.

### VPN Role

This node hosts the WireGuard VPN service and acts as a secure ingress path into the LAN.

- Listens on UDP `51820`
- Receives forwarded traffic from the router
- Performs NAT to allow VPN clients to access LAN resources without requiring router-level routing changes

## Backlight / Lid Behavior

This node uses the unified lid-aware backlight control model documented in:

```text
docs/services/backlight-control.md
```

Expected behavior:

- Lid closed → backlight turns off
- Lid open → no automatic change
- Reboot with lid open → screen stays visible for recovery
- SSH recovery available with `sudo backlight-on`

Installed standard scripts:

```text
/usr/local/bin/backlight-off
/usr/local/bin/backlight-on
```

ACPI lid rule:

```text
/etc/acpi/events/lid-close
```

## Notes

- This machine is intentionally tracked as a separate node rather than folded into the primary server documentation.
- The primary production-ish home server remains `ubuntu` at `192.168.86.53`.
- Do not assume services are running on this node unless they are explicitly documented here or in service-specific docs.
- - This node is now the primary trusted ingress point for remote access via VPN.
