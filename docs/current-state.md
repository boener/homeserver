# 🧠 Home Server – Current State

_Last updated: 2026-04-20_

---

## 💽 Backup System

### Overview
The system uses a **two-layer backup architecture** with explicit directory separation:

- `/mnt/backup/storage` → data backups (user)
- `/mnt/backup/system` → system backups (root)

This prevents accidental cross-deletion and keeps responsibilities clearly isolated.

---

## 📦 Data Backup (User)

### Scope
- `/mnt/storage`

### Ownership
- Runs as user `ian`

### Structure
```
/mnt/backup/storage/
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
- `rsync -a --delete` for system files
- snapshot via `cp -al`
- replaces same-day snapshot if rerun
- retention: ~14 days

### Automation
```
30 3 * * * /usr/local/sbin/system-backup.sh
```

---

## 🧠 Design Notes

### Why the `storage/` subdirectory was introduced
Originally, data backups lived directly under `/mnt/backup/`, which made it easy to:
- accidentally delete the entire backup tree
- mix system and data backup concerns

Moving to:
```
/mnt/backup/storage/
```
creates a clear boundary and reduces risk.

### Failure Mode Prevented
- accidental deletion of `/mnt/backup/*` no longer destroys both backup systems
- system backups remain intact even if storage backup is misconfigured

---

## 🧾 Changelog

### 2026-04-20
- Refined backup structure to separate `/mnt/backup/storage` and `/mnt/backup/system`
- Updated storage backup script to use new path
- Improved safety against accidental deletion

