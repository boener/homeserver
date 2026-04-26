# 💽 Backup System

_Last updated: 2026-04-26_

---

## Overview

The primary server (`ubuntu`) uses a local external Toshiba backup drive mounted at:

```text
/mnt/backup
```

Backups are stored on that drive in two main areas:

```text
/mnt/backup/storage
/mnt/backup/system
```

The backup system currently uses simple shell scripts plus cron. It does not use a dedicated backup application such as Borg, Restic, or Timeshift.

---

## Backup Drive Mounting

The backup drive is mounted by UUID in `/etc/fstab`.

Current backup drive UUID:

```text
1c7d81b7-72b7-407a-b5ca-fcc23bff725c
```

Current `/etc/fstab` behavior:

```text
UUID=1c7d81b7-72b7-407a-b5ca-fcc23bff725c /mnt/backup ext4 defaults,nofail,x-systemd.automount 0 2
```

### Why automount is used

The backup drive is an external USB drive. USB storage can sometimes appear slightly late during boot. A normal boot-time mount may fail if the drive is not ready yet.

The current design uses:

- `nofail` — boot continues even if the backup drive is unavailable
- `x-systemd.automount` — systemd watches `/mnt/backup` and mounts the drive on first access

This means backup scripts can trigger the mount simply by accessing `/mnt/backup`.

### Verification commands

Check whether the automount unit is active:

```bash
systemctl status mnt-backup.automount
```

Expected healthy state:

```text
Active: active (waiting)
```

Check whether the real drive is mounted:

```bash
findmnt /mnt/backup
```

Expected mounted source:

```text
/dev/sdc1 ext4 ... /mnt/backup
```

The device name may change across boots, but the UUID should remain stable.

---

## Storage Backup

### Script location

```text
/home/ian/backup.sh
```

### Cron schedule

This runs from user `ian`'s crontab:

```text
0 3 * * * /home/ian/backup.sh
```

Schedule meaning:

- every day
- at 3:00 AM
- as user `ian`

### Source and destination

Source:

```text
/mnt/storage/
```

Destination:

```text
/mnt/backup/storage
```

Expected layout:

```text
/mnt/backup/storage/current
/mnt/backup/storage/snapshots
```

### Behavior

The script maintains:

- `current` — latest synchronized copy of `/mnt/storage/`
- `snapshots/YYYY-MM-DD` — dated hard-link snapshot copies

The storage backup uses `rsync` and hard links:

- `rsync -a --delete --link-dest="$CURRENT"` updates the current copy efficiently
- `cp -al` creates a dated snapshot using hard links
- snapshots older than 14 days are deleted

### Safety check

The script includes a mount check before writing:

```bash
if ! mountpoint -q /mnt/backup; then
  echo "Backup drive not mounted. Exiting."
  exit 1
fi
```

This prevents the script from accidentally writing backup data into the plain `/mnt/backup` directory on the root filesystem if the external drive is not mounted.

---

## System Backup

### Script location

```text
/usr/local/sbin/system-backup.sh
```

### Cron schedule

This runs from root's crontab:

```text
30 3 * * * /usr/local/sbin/system-backup.sh
```

Schedule meaning:

- every day
- at 3:30 AM
- as root

### Source and destination

Sources:

```text
/etc/
/home/ian/
```

Destination:

```text
/mnt/backup/system
```

Expected layout:

```text
/mnt/backup/system/current/etc
/mnt/backup/system/current/home-ian
/mnt/backup/system/snapshots
```

### Behavior

The script maintains:

- `current/etc` — latest synchronized copy of `/etc/`
- `current/home-ian` — latest synchronized copy of `/home/ian/`
- `snapshots/YYYY-MM-DD` — dated hard-link snapshot copies

The system backup uses:

- `rsync -a --delete` to keep the current copy aligned with the source directories
- `cp -al` to create dated hard-link snapshots
- a 14-day retention window for dated snapshots

The script uses `set -e`, so it exits immediately if a command fails.

### Safety check

The script includes the same mount check as the storage backup:

```bash
if ! mountpoint -q /mnt/backup; then
  echo "Backup drive not mounted. Exiting."
  exit 1
fi
```

This prevents root's system backup from writing into the unmounted `/mnt/backup` directory on the root filesystem.

---

## Operational Notes

### Manual test commands

Run the storage backup manually:

```bash
/home/ian/backup.sh
```

Run the system backup manually:

```bash
sudo /usr/local/sbin/system-backup.sh
```

Check the backup drive mount:

```bash
findmnt /mnt/backup
```

Check backup disk usage:

```bash
df -h /mnt/backup
```

### Known previous failure mode

A previous failure occurred when `/mnt/backup` was not mounted. The storage backup script continued running and wrote data into the underlying `/mnt/backup` directory on the root filesystem instead of the external drive.

That accidental root-disk backup data was removed, the storage directory layout was corrected, and both backup scripts now include explicit mount safety checks.

### Future improvement

Cron output is not currently redirected to a dedicated backup log file. A useful future improvement would be to redirect each backup job to a log, for example:

```text
0 3 * * * /home/ian/backup.sh >> /home/ian/backup.log 2>&1
```

and a similar root-owned log for the system backup.
