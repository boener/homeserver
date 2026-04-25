# Backlight Control

_Last updated: 2026-04-25_

## Overview

Both lab laptops now use the same lid-aware backlight control model.

The goal is to reduce heat and power draw from closed laptop screens while preserving an easy recovery path.

## Standard Behavior

- Lid open: screen may remain on
- Lid closed: backlight turns off
- Reboot with lid open: screen is visible again for recovery
- Manual SSH recovery is available with `sudo backlight-on`

This replaces the older drifted setup where one machine used a boot-delay systemd service and the MacBook used a MacBook-specific ACPI script.

## Machines Covered

### ubuntu

- Ubuntu 24.04 Server
- Uses `/sys/class/backlight/intel_backlight/`
- Previous boot-delay `screenoff.service` was removed
- Previous `/usr/local/bin/screenoff` and `/usr/local/bin/screenon` scripts were removed
- `acpid` was installed to support lid events

### macbook

- Ubuntu 24.04 Server
- Uses `/sys/class/backlight/acpi_video0/`
- Previous `/usr/local/bin/lid-close-screen-off.sh` script was removed
- Previous `/etc/acpi/events/lid-close` rule was recreated to point at the standard script
- Previous `~/screen` manual recovery script was removed

## Standard Scripts

Both machines now use the same script names and logic.

### `/usr/local/bin/backlight-off`

```bash
#!/bin/bash

BACKLIGHT_DIR="$(find /sys/class/backlight -mindepth 1 -maxdepth 1 -type l | head -n 1)"

if [ -z "$BACKLIGHT_DIR" ]; then
    exit 1
fi

echo 0 > "$BACKLIGHT_DIR/brightness"
```

### `/usr/local/bin/backlight-on`

```bash
#!/bin/bash

BACKLIGHT_DIR="$(find /sys/class/backlight -mindepth 1 -maxdepth 1 -type l | head -n 1)"

if [ -z "$BACKLIGHT_DIR" ]; then
    exit 1
fi

MAX=$(cat "$BACKLIGHT_DIR/max_brightness")

echo "$MAX" > "$BACKLIGHT_DIR/brightness"
```

Both scripts are owned by root and executable:

```text
-rwxr-xr-x root root /usr/local/bin/backlight-off
-rwxr-xr-x root root /usr/local/bin/backlight-on
```

## ACPI Lid Event

Both machines use the same ACPI event rule:

```text
/etc/acpi/events/lid-close
```

Contents:

```text
event=button/lid.*
action=/usr/local/bin/backlight-off
```

After changes to ACPI event files, reload with:

```bash
sudo systemctl restart acpid
```

## Verification

Confirm lid state:

```bash
cat /proc/acpi/button/lid/*/state
```

Expected output:

```text
state:      open
```

or:

```text
state:      closed
```

Confirm ACPI events fire:

```bash
acpi_listen
```

Expected lid event output:

```text
button/lid LID open
button/lid LID close
```

Exit with `Ctrl+C`.

Confirm installed files:

```bash
ls -l /usr/local/bin/backlight-* /etc/acpi/events/lid-close
```

## Recovery

To turn the screen back on over SSH:

```bash
sudo backlight-on
```

To turn it off manually:

```bash
sudo backlight-off
```

## Design Rationale

The hardware backlight path differs by machine, so the scripts intentionally autodetect the available backlight interface instead of hardcoding `intel_backlight` or `acpi_video0`.

This keeps the behavior standardized while still respecting each machine's kernel/firmware-specific backlight implementation.
