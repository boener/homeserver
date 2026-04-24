# Cowrie SSH Honeypot

_Last updated: 2026-04-24_

---

## Purpose

Cowrie is running as an SSH honeypot to capture bot login attempts and interactive fake-shell behavior.

The goal is to observe what automated attackers do after they believe they have reached a weak Linux SSH server.

This is intentionally separate from the server's real SSH service.

---

## Current Status

Cowrie is installed and running in Docker.

It has been tested locally and from another LAN client.

Confirmed behavior:

- SSH login prompt is presented on the honeypot port
- fake Debian-style shell is shown after successful honeypot login
- commands typed in the fake shell are logged
- failed commands are logged
- JSON logs persist on the host filesystem

---

## Docker Details

Image:

```text
cowrie/cowrie:latest
```

Container name:

```text
cowrie
```

Restart policy:

```text
unless-stopped
```

Port mapping:

```text
host 2222 -> container 2222/tcp
```

Docker currently also exposes Cowrie's internal `2223/tcp` inside the container, but only SSH on host port `2222` is intentionally published.

---

## Host Paths

Project directory:

```text
/home/ian/cowrie
```

Persistent log directory:

```text
/home/ian/cowrie/log
```

Primary JSON event log:

```text
/home/ian/cowrie/log/cowrie.json
```

Persistent downloads directory:

```text
/home/ian/cowrie/downloads
```

Persistent TTY/session recording directory:

```text
/home/ian/cowrie/tty
```

---

## Current Run Command

```bash
docker run -d \
  --name cowrie \
  --restart unless-stopped \
  -p 2222:2222 \
  -v ~/cowrie/log:/cowrie/cowrie-git/var/log/cowrie \
  -v ~/cowrie/downloads:/cowrie/cowrie-git/var/lib/cowrie/downloads \
  -v ~/cowrie/tty:/cowrie/cowrie-git/var/lib/cowrie/tty \
  cowrie/cowrie:latest
```

---

## Permissions Note

The persistent folders currently use permissive permissions so the container user can write logs and session recordings:

```bash
chmod -R 777 ~/cowrie/log ~/cowrie/downloads ~/cowrie/tty
```

This solved container write failures such as:

```text
PermissionError: [Errno 13] Permission denied: 'var/log/cowrie/cowrie.json'
PermissionError: [Errno 13] Permission denied: 'var/lib/cowrie/tty/...log'
```

This is acceptable for the current hobby honeypot phase, but it is not a refined long-term permissions model.

Future cleanup should identify Cowrie's container UID/GID and assign narrower ownership/permissions.

---

## Known Setup Pitfall

Do not blindly mount over Cowrie's whole internal `etc` directory or broad internal runtime paths.

An early attempt mounted host directories over Cowrie internals and caused Cowrie to lose access to required built-in fake filesystem data, producing errors such as:

```text
FileNotFoundError: /cowrie/cowrie-git/src/cowrie/data/honeyfs/etc/passwd
```

The current setup only bind-mounts the useful runtime output directories:

- logs
- downloads
- tty recordings

---

## Testing Commands

From the server itself:

```bash
ssh -p 2222 root@localhost
```

From another LAN machine:

```bash
ssh -p 2222 root@192.168.86.53
```

Known working test credentials:

```text
username: root
password: password
```

Example fake-shell commands confirmed in logs:

```bash
whoami
ls
uname -a
ver
Random commands LOLOL
```

---

## Log Inspection

Show recent JSON events:

```bash
cat ~/cowrie/log/cowrie.json | tail -n 10
```

Show Docker logs:

```bash
docker logs cowrie --tail 50
```

Expected event types include:

```text
cowrie.login.success
cowrie.command.input
cowrie.command.failed
cowrie.client.size
cowrie.session.params
```

---

## Security Boundary

Real SSH is not exposed through Cowrie.

Current safety model:

- Cowrie listens on host port `2222`
- real SSH remains separate from the honeypot
- router port forwarding to Cowrie has not been configured yet
- no firewall or router changes have been made for this service
- external exposure should only happen after explicit approval

Potential future exposure model:

```text
Internet TCP/22 -> router port forward -> server 192.168.86.53:2222 -> Cowrie container 2222
```

That router forwarding is not currently active.

---

## Website Integration Notes

The future scrolling website/interface should consume:

```text
/home/ian/cowrie/log/cowrie.json
```

The JSON log is newline-delimited JSON, with one event per line.

Useful fields for display:

- `timestamp`
- `eventid`
- `src_ip`
- `username`
- `password`
- `input`
- `message`
- `session`

Command events are especially useful for the planned fake-terminal style display.
