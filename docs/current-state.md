# 🧠 Home Server – Current State

_Last updated: 2026-04-20_

---

## 🧠 Purpose of This Document

This file is the **canonical, current-state snapshot** of the home server.

It is intentionally:
- Detailed (so concepts are preserved)
- Accurate (reflects real system state)
- Updated over time (acts as living memory)

---

## 🖥️ Hardware

### System
- Dell Inspiron 5570 laptop
- Intel i5-8250U (4 cores / 8 threads)
- 16GB RAM

### Storage
- 512GB SSD → OS (`/`)
- 1TB HDD → primary data (`/mnt/storage`)
- 1TB external USB drive → local backup target (`/mnt/backup`)

### Networking
- ❌ Built-in NIC (enp1s0): 10/100 (configured, normally disconnected)
- ✅ Active NIC: USB 3.0 Gigabit (`enx6c6e072bdc14`)

---

## 💽 Backup System

### Overview
The system uses a **two-layer backup architecture**:

1. **Data Backup (user-level)**
2. **System Backup (root-level)**

Both use the same snapshot model but run under different privilege levels.

---

## 📦 Data Backup (User)

### Scope
- `/mnt/storage`

### Ownership
- Runs as user `ian`

### Structure
```
/mnt/backup/
├── current/
└── snapshots/YYYY-MM-DD/
```

### Script
- `/home/ian/backup.sh`

### Behavior
- `rsync -a --delete` syncs data into `current/`
- `cp -al` creates snapshot using hard links
- retention: ~14 days

### Automation
- Cron (user):
```
0 3 * * * /home/ian/backup.sh
```

---

## ⚙️ System Backup (Root)

### Scope
- `/etc`
- `/home/ian`

### Ownership
- Runs as `root`

### Why Root Is Required
System files are owned by root and protected from hard-link creation by non-root users.
Running as root avoids permission errors and eliminates the need for fragile exclude lists.

### Structure
```
/mnt/backup/system/
├── current/
│   ├── etc/
│   └── home-ian/
└── snapshots/YYYY-MM-DD/
```

### Script
- `/usr/local/sbin/system-backup.sh`

### Behavior
- `rsync -a --delete` for both `/etc` and `/home/ian`
- snapshot created via `cp -al`
- existing same-day snapshot is replaced if script reruns
- retention: ~14 days

### Automation
- Cron (root):
```
30 3 * * * /usr/local/sbin/system-backup.sh
```

### Timing Design
- Data backup runs at 3:00 AM
- System backup runs at 3:30 AM
- Prevents disk contention and overlapping IO spikes

---

## 🧠 Design Rationale

This system intentionally separates concerns:

### Data vs System
- Data is large, user-owned, and frequently changing
- System config is small, critical, and permission-sensitive

### Privilege Separation
- User backup avoids unnecessary root usage
- System backup uses root where required

### Recovery Model

This enables three recovery paths:

#### File-level restore
```
/mnt/backup/system/snapshots/YYYY-MM-DD/
```

#### System rebuild
- reinstall Ubuntu
- restore `/etc` and `/home/ian`

#### Data restore
```
/mnt/backup/current/
```

---

## ⚠️ Limitations

This is a **single-location backup system**:

Protected:
- disk failure
- accidental deletion
- configuration mistakes

Not protected:
- theft
- fire
- catastrophic system loss
- simultaneous failure of both disks

---

## 🧾 Changelog

### 2026-04-20
- Added dual-layer backup system (data + system)
- Introduced root-level system backup script
- Added `/etc` and `/home/ian` to backup scope
- Implemented privilege-separated backup design
- Added staggered cron scheduling (3:00 / 3:30)

### 2026-04-19
- Initial documentation system created
