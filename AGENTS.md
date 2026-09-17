# CLAUDE.md

This file provides guidance to LLM agents when working with code in this repository.

## What this is

Bash maintenance scripts for a Raspberry Pi (Pi-hole host running Docker dashboards). No build, no tests, no linter configured. Scripts target Debian/Raspberry Pi OS and are not runnable on macOS (they call `apt-get`, `pihole`, `dhclient`, `/sys/class/thermal`).

## Commands

```
./script/setup            # first-time install: runs script/bootstrap then bin/setup-crons
./script/bootstrap        # symlinks ~/bin -> ~/code/piscripts/bin, installs ghostty terminfo system-wide
bin/setup-crons           # idempotently (re)writes the crontab entries
```

Cron jobs installed by `bin/setup-crons` (each piped to `logger --tag <name>`):

| Schedule | Script | Guard |
| --- | --- | --- |
| Sat 00:00 | `bin/update-system` | `command -v update-system` |
| Fri 00:00 | `bin/update-pihole` | `command -v pihole` |
| 1st of month 02:00 | `bin/monthly-reboot` | `command -v monthly-reboot` |

## Structure and conventions

- `bin/` is the whole product. `script/bootstrap` symlinks it to `~/bin`, so scripts are invoked by bare name and `setup-crons` guards on `command -v <script>` — a new cron entry will silently not install until the symlink is in `PATH`.
- `setup-crons` is idempotent by `grep -v "bin/<name>"` then re-append. Add new crons with the same pattern.
- Scripts use `#!/bin/bash` and `set -euo pipefail` with the explanatory comment block copied verbatim from existing files (`set -e` only where `-u`/pipefail would break, e.g. `setup-crons`, `update-pihole`). Match that header in new scripts.
- `bin/update-pihole` redirects all output to `logger` via `exec 1> >(logger -s -t ...)`; the other cron scripts rely on the crontab pipe to `logger` instead.
- `bin/print-and-run` and `bin/colors` are libraries meant to be sourced, not run.
- `bin/temperature` is the one Python script (`/usr/bin/python`).
- Repo path `~/code/piscripts` is hardcoded in `script/bootstrap`; `setup-crons` uses `$(pwd)` after `cd` to the repo root.
