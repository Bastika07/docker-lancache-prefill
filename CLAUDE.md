# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Docker image (optimized for Unraid) that downloads, auto-updates, and runs
[tpill90's LANCache prefill tools](https://github.com/tpill90) — BattleNetPrefill,
EpicPrefill, and SteamPrefill — on cron schedules. There is no application code,
build system, or test suite; the repo is a Dockerfile plus bash scripts.

## Commands

```bash
# Build the image
docker build -t lancache-prefill .

# Run (see README.md for the full env-var example)
docker run --name LANCache-Prefill -d \
  --env 'ENABLE_STEAM=true' \
  --volume /path/to/lancacheprefill:/lancacheprefill \
  lancache-prefill

# Lint shell scripts
shellcheck scripts/*.sh cron/*.sh
```

## Architecture

Startup flow: `scripts/start.sh` (ENTRYPOINT, runs as root) → sets UID/GID/umask,
runs optional `/opt/scripts/user.sh`, starts the cron daemon, fixes ownership, then
runs `scripts/start-server.sh` as the unprivileged `prefill` user (`$USER`).

`scripts/start-server.sh` does, for each enabled prefill tool (gated by
`ENABLE_BN`/`ENABLE_EPIC`/`ENABLE_STEAM`):

1. **Install/update**: fetches the latest release tag from the GitHub API, downloads
   the linux-x64 zip into `$DATA_DIR/<Tool>Prefill/`. The installed version is tracked
   by an empty marker file in `$DATA_DIR` (e.g. `steamprefill_v2.x.x`) — comparing the
   marker name to the latest tag drives the update check (`UPDATES=true`).
2. **Config check**: if the tool's per-tool config file (e.g.
   `SteamPrefill/Config/account.config`) is missing, the tool is disabled for this run
   and an ATTENTION banner tells the user to run `select-apps` interactively.
3. **Cron setup**: writes `/tmp/cron` and installs it via `crontab`. If
   `CRON_SCHED_GLOBAL` is set it overrides the per-tool schedules
   (`CRON_SCHED_BN`/`CRON_SCHED_EPIC`/`CRON_SCHED_STEAM`) and runs
   `cron/global_prefill.sh` (all tools sequentially) instead of the per-tool
   `cron/*_prefill.sh` scripts.
4. **Log tailing**: the container stays alive by `tail -f` on the per-tool log files
   in `$DATA_DIR/logs/`, which is how prefill output reaches the container log.

Env vars needed by the cron jobs are exported to `/opt/cron/env.sh` (sourced by every
cron script), since cron doesn't inherit the container environment.

The three per-tool cron scripts and the three install/update blocks in
`start-server.sh` are deliberately parallel copies — when changing one tool's logic,
apply the same change to the other two.

## Conventions

- All defaults and env vars are declared in the Dockerfile (`ENV` lines); the README
  documents them in tables. Keep Dockerfile, scripts, and README in sync when adding
  or changing a variable.
- Cron scripts guard against overlapping runs with `pidof <Tool>Prefill`.
- Unrecoverable startup errors (failed download, missing version) put the container
  into `sleep infinity` rather than exiting, so the user can inspect it.
