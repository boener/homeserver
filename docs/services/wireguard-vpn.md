# WireGuard VPN (MacBook Node)

_Last updated: 2026-04-29_

## Overview

WireGuard is running on the `macbook` node to provide full-tunnel VPN access into the home network.

Because the home LAN and some remote networks can both use `192.168.86.0/24`, direct VPN access to LAN IPs may fail due to subnet collision. The current practical workaround is to use the MacBook's WireGuard address (`10.0.0.1`) as the stable remote entry point.

## Host

- macbook (192.168.86.45)
- interface: enp2s0f0
- vpn interface: wg0

## Network

- subnet: 10.0.0.0/24
- server: 10.0.0.1
- client: 10.0.0.2

## Port

- UDP 51820 (forwarded on router)

## DNS

- Pi-hole at 192.168.86.49

## Endpoint

- ianboen.com:51820 (Cloudflare DNS-only)

## Service

- wg-quick@wg0
- enabled at boot and reboot-tested

## Remote Access Pattern

### SSH

From a VPN client, SSH access to the MacBook works through the WireGuard interface:

```bash
ssh ian@10.0.0.1
```

If another LAN host is needed, the MacBook can be used as an internal jump point.

### SFTP / WinSCP

SFTP access to the MacBook works over the same SSH path:

```bash
sftp ian@10.0.0.1
```

On Windows, WinSCP can connect with:

- Protocol: SFTP
- Host: `10.0.0.1`
- Port: `22`
- User: `ian`

This provides remote file access without changing the home LAN subnet.

### Ubuntu Storage Bridge

The MacBook has `sshfs` installed and can manually mount the primary Ubuntu server's storage into the VPN-accessible share folder:

```bash
sshfs ian@192.168.86.53:/mnt/storage ~/vpn-share/storage
```

Resulting path from WinSCP:

```text
/home/ian/vpn-share/storage
```

This path exposes Ubuntu's `/mnt/storage` through the MacBook over the VPN. A small file write test from a remote Windows machine succeeded.

This is currently a manual mount. It is not yet configured to persist across reboot.

## Verification

```bash
sudo wg show
sudo ss -ulnp | grep 51820
ip addr show wg0
sftp ian@10.0.0.1
ls ~/vpn-share/storage
```

## Failure Modes

- No handshake → check port forward / DNS
- Handshake but no LAN access → check iptables MASQUERADE, IP forwarding, and possible subnet collision
- Connected but no internet → check AllowedIPs (0.0.0.0/0)
- Can SSH/SFTP to `10.0.0.1` but cannot access LAN IPs → likely remote/home subnet collision
- `~/vpn-share/storage` is empty or inaccessible → SSHFS mount may not be active

## Notes

- Runs on secondary node to isolate network services
- Full tunnel routing is enabled
- NAT is used to allow LAN access without router changes when subnets do not collide
- Cowrie on port 22 is unaffected
- Current file-access workaround avoids changing the home LAN subnet and avoids disrupting IoT/roommate devices

## Future Work

- consolidate key storage into /etc/wireguard
- add additional clients
- decide whether to automate the SSHFS mount or keep it manual
- consider long-term LAN subnet migration only during a planned maintenance window
