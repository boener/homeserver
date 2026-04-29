# WireGuard VPN (MacBook Node)

_Last updated: 2026-04-29_

## Overview

WireGuard is running on the `macbook` node to provide full-tunnel VPN access into the home network.

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

## Verification

sudo wg show
sudo ss -ulnp | grep 51820

## Notes

- Runs on secondary node to isolate network services
- Full tunnel routing is enabled
- NAT is used to allow LAN access without router changes
- Cowrie on port 22 is unaffected

## Future Work

- consolidate key storage into /etc/wireguard
- add additional clients
