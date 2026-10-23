# CLAUDE.md

This file provides guidance to LLM agents when working with code in this repository.

## What this is

Bash maintenance scripts for a Raspberry Pi (Pi-hole host running Docker dashboards). No build, no tests. Scripts target Raspberry Pi OS Bullseye (python3 only, dhcpcd, iproute2, Docker Compose v2 `docker compose`) and are not runnable on macOS (they call `apt-get`, `pihole`, `dhcpcd`, `/sys/class/thermal`).

## Commands

```
./script/setup            # first-time install: runs script/bootstrap then bin/setup-crons
./script/bootstrap        # symlinks ~/bin -> <repo>/bin (idempotent), installs ghostty terminfo system-wide
bin/setup-crons           # idempotently (re)writes the crontab entries
bin/setup-unbound         # optional, run once: installs unbound on 127.0.0.1#5335 and sets it as Pi-hole's upstream
bin/setup-tailscale       # optional, run once: installs Tailscale, enables tailscaled at boot with Restart=always
shellcheck bin/* script/* # lint (what CI runs; bin/temperature is Python, use python3 -m py_compile)
```

CI (`.github/workflows/lint.yml`) runs shellcheck on every `#!/bin/bash` file under `bin/` and `script/` plus `py_compile` on `bin/temperature`.

Cron jobs installed by `bin/setup-crons` (each piped to `logger --tag <name>`):

| Schedule | Script | Guard |
| --- | --- | --- |
| Sat 00:00 | `bin/update-system` | `[ -x bin/update-system ]` |
| Fri 00:00 | `bin/update-pihole` | `command -v pihole` |
| 1st of month 02:00 | `bin/monthly-reboot` | `[ -x bin/monthly-reboot ]` |

## Structure and conventions

- `bin/` is the whole product. `script/bootstrap` symlinks it to `~/bin` (refuses to clobber a real `~/bin` directory), so scripts are invoked by bare name.
- `setup-crons` is idempotent via `add_cron "<schedule>" "<name>"` (`grep -v "bin/<name>"` then re-append, output piped to `logger --tag <name>`). Add new crons through that function.
- Entry-point scripts use `#!/bin/bash` and `set -euo pipefail` with the explanatory comment block copied verbatim from existing files (`set -e` only where `-u`/pipefail would break, e.g. `setup-crons`, `update-pihole`). Match that header in new scripts.
- `bin/update-pihole` redirects all output to `logger` via `exec 1> >(logger -s -t ...)`; the other cron scripts rely on the crontab pipe to `logger` instead. (Because `-s` echoes to stderr, pihole lines may land in syslog twice — known, tolerated.)
- `bin/print-and-run` and `bin/colors` are libraries meant to be sourced, not run. They deliberately have no `set -euo pipefail` (it would leak into the caller's shell). Nothing in the repo sources them yet.
- `bin/setup-unbound` is manual/opt-in (not called by `script/setup`, no cron). It renders `templates/unbound/pi-hole.conf` (verbatim from the Pi-hole guide, `@INTERFACE@`/`@PORT@` placeholders) to `/etc/unbound/unbound.conf.d/pi-hole.conf` via `sed | sudo tee`, and sets the upstream via `pihole-FTL --config dns.upstreams` (Pi-hole v6).
- `bin/setup-tailscale` is manual/opt-in (not called by `script/setup`, no cron). It installs via the official `tailscale.com/install.sh`, writes the systemd drop-in `/etc/systemd/system/tailscaled.service.d/override.conf` (`Restart=always`, `After`/`Wants=network-online.target`), and runs `tailscale up --accept-dns=false` so `/etc/resolv.conf` is never rewritten and Pi-hole stays the host resolver. Driven by env vars `TS_AUTHKEY`, `TS_HOSTNAME`, `TS_ROUTES`, `TS_EXIT_NODE`, `TS_PIHOLE_DNS`. Enables IP forwarding via `/etc/sysctl.d/99-tailscale.conf` only when routes or an exit node are requested. Idempotent: re-runs use `tailscale set` once `BackendState` is `Running`.
- `bin/temperature` is the one Python script (`#!/usr/bin/env python3`).
- `bin/local-ip` and `bin/renew-dhcp` take an optional interface arg, default `eth0`. `renew-dhcp` dispatches on the active service: `dhcpcd --rebind` (Bullseye) or `nmcli connection up` (Bookworm+ NetworkManager). Note the Pi's `eth0` is a static address in `dhcpcd.conf`, so rebind is a no-op there.
- Both `script/bootstrap` and `bin/setup-crons` locate the repo via `cd "$(dirname "$0")/.."` + `$(pwd)`; nothing hardcodes the clone path.
