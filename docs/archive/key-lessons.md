# 🧠 Key Lessons & Non-Obvious Knowledge

This document captures **high-value insights, gotchas, and reusable patterns** discovered while building the home server.

These are not step-by-step instructions—they are things that are easy to forget but extremely useful later.

---

## 🪟 Windows Networking Weirdness

### SMB Issues
- Windows often blocks **guest SMB access** by default
- NTLM / credential conflicts can silently break connections

### Routing Issues
- Virtual adapters (e.g., Hyper-V `vEthernet`) can hijack routing
- Symptoms:
  - Cannot reach server
  - Strange timeouts

### Fix Pattern
- Disable conflicting adapters
- Or adjust interface metrics

---

## 🔍 Advanced Debugging Pattern

### Isolate Variables
Always test in layers:
1. DNS resolution
2. Raw HTTP
3. Port forwarding
4. TLS / HTTPS

### Useful Command

```bash
curl -vk --resolve boener.duckdns.org:443:127.0.0.1 https://boener.duckdns.org/
```

This forces:
- local routing
- real TLS handshake
- certificate validation

Extremely useful for diagnosing HTTPS issues.

---

## 📶 Disabling Hardware via Driver Blacklisting

Example (WiFi):

```bash
echo "blacklist ath10k_pci" | sudo tee /etc/modprobe.d/blacklist-wifi.conf
```

### Insight
- Disables device at kernel level
- More reliable than just "disconnecting"

### Reversal
```bash
sudo rm /etc/modprobe.d/blacklist-wifi.conf
sudo reboot
```

---

## ⚙️ systemd Service Pattern

Reusable structure for running custom apps:

```ini
[Unit]
Description=My Service
After=network.target

[Service]
User=ian
WorkingDirectory=/path/to/app
ExecStart=/path/to/venv/bin/python app.py
Restart=always

[Install]
WantedBy=multi-user.target
```

### Insight
- Works for **any long-running process**, not just Flask
- Ensures:
  - auto-start on boot
  - restart on crash

---

## 🌐 Split-Horizon DNS (Critical Concept)

Same domain name, different IPs depending on location:

| Location        | Resolves to         |
|----------------|---------------------|
| LAN            | 192.168.x.x         |
| Public Internet| WAN IP              |

### Why This Matters
- Avoids NAT loopback issues
- Keeps URLs consistent everywhere
- Enables clean HTTPS usage

---

## 🔐 TLS / HTTPS Insight

- TLS is **hostname-based**, not IP-based
- Certificates are issued for domains

### Implication
- Always access services via domain
- Not via raw IP

---

## 🖥️ Hardware Reality Check

- Consumer laptop hardware is:
  - not redundant
  - not designed for 24/7 uptime

### Specific Constraints
- Built-in NIC limited to 10/100
- Single HDD = no redundancy

### Insight

> Performance and reliability limits are often hardware-bound, not software-bound.

---

## 🧠 System Design Pattern Learned

You are effectively running:

- Reverse proxy (Caddy)
- Internal services (Flask, Jellyfin)
- DNS control (Pi-hole)

### Core Pattern

```
Client → DNS → Reverse Proxy → Service
```

This pattern scales to:
- multiple services
- access control
- public + private separation

---

## 🎯 Philosophy

> Don’t memorize commands.
> Understand systems.

The value of this project is not the setup—it is the mental models gained:
- networking layers
- service boundaries
- debugging methodology

---

## 📌 Purpose of This File

This file exists to preserve:
- hard-earned insights
- non-obvious fixes
- reusable mental models

Not to duplicate documentation that already exists elsewhere.

