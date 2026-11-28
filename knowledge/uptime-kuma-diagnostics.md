---
topic: uptime-kuma-diagnostics
category: knowledge
tags: [knowledge, uptime-kuma-diagnostics, uptime-kuma, komodo, pi-studio]
updated_at: 2026-11-28T07:58:26.581697+00:00
compacted_at: 2026-11-28T07:58:26.581697+00:00
confidence: 0.95
---

# Knowledge: Uptime-Kuma-Diagnostics

## Diagnostics & Alerts

- **Simultaneous 48,000ms timeouts on `.klinker-cabin.computer` monitors**:
  Uptime Kuma (`uptime.klinker.computer`) runs on `pi-studio` in Boulder and
  probes Cabin services (`unraid-nas`, `unraid-seedbox`, `unraid-aibox`) in
  Estes over the inter-site WAN / `gluetun-home` WireGuard link. Simultaneous
  `timeout of 48000ms exceeded` alerts while local containers and ingress are
  healthy indicate either:
  - A transient inter-site WAN or WireGuard connectivity drop.
  - CrowdSec on `unraid-nas` banning the Boulder home WAN IP (shared by
    `pi-studio` and user devices) and `cs-unifi-bouncer` adding it to the Cabin
    UniFi firewall drop group—for example, `crowdsecurity/http-probing`
    triggering on bursts of `404` responses like missing
    `GET /Audio/<uuid>/Lyrics` requests from Jellyfin clients. Inspect bans on
    `unraid-nas` with `docker exec crowdsec cscli decisions list`, remove false
    positives with `docker exec crowdsec cscli decisions delete --ip <ip>`,
    and verify removal in `cs-unifi-bouncer` logs.
- **`kuma_get_monitors` `HTTP 502 Bad Gateway` vs `HTTP 0`**: A `502` (rather
  than `HTTP 0` timeout) confirms `pi-studio` and `pi-studio-reverse-proxy`
  (`openresty`) are online and reachable over WAN, isolating the failure to
  the upstream `uptime-kuma` container (`pi-studio-dashboard` stack)
  restarting or initializing.
- **06:34–06:35 MST `ECONNREFUSED` spikes**: Scheduled maintenance/backup
  windows simultaneously restart containers across `unraid-nas` (including
  `nginx-proxy-manager` terminating HTTPS on `38.75.214.203:443`) and
  `unraid-seedbox`, causing transient `ECONNREFUSED` alerts on
  `Seedbox Notifications`, `NAS Notifications`, `qui`, `Cabin`, and
  `Upload Assistant` that self-recover within a couple of minutes.
- **Stale `gluetun` network references after auto-updates**: On `unraid-nas`,
  `semaphore-boulder` (`Ansible Boulder`) uses
  `network_mode: service:gluetun-home` (Komodo `management` stack) and
  `rsync-backup` uses `network_mode: service:gluetun-home-backup` (Komodo
  `files` stack). Recreating `gluetun-home` or `gluetun-home-backup` (e.g.,
  during the 3:00 AM container auto-update window) leaves dependents pinned to
  the destroyed container ID, causing `Ansible Boulder` to fail with HTTP 502
  and `komodo_restart_container` to fail with `No such container: <old_id>`.
  Recreate the dependent container via:

  ```bash
  docker exec \
    -w /mnt/user/docker/komodo/periphery/stacks/nas-management/stacks/unraid-nas/management \
    komodo-periphery docker compose -p management --env-file .env \
    up -d --force-recreate semaphore-boulder
  ```

## Configuration & Maintenance

- **API vs Socket.IO authentication**: `UPTIME_KUMA_API_KEY` only
  authenticates the read-only Prometheus `/metrics` endpoint. Mutating
  monitors or status pages (`deleteMonitor`, `saveStatusPage`) requires
  authenticating via Uptime Kuma's Socket.IO WebSocket with an admin username
  and password.
- **JSONata thresholds in `HTTP(s) - Json Query` monitors**: JSONata
  expressions (`$number`, `$replace`, ternary `? :`) parse and threshold-check
  string-formatted metrics (such as percentages) directly without modifying
  the upstream API. Using a ternary expression with Expected Value `ok`
  ensures failed checks include the offending value (`value was: [49%]`)
  instead of `false`:

  ```text
  $number($replace($[2].estes_battery, "%", "")) >= 50 ? "ok" : $[2].estes_battery
  ```

- **Compacting `kuma.db` on SD card storage**: When pruning millions of
  `heartbeat` rows or compacting a multi-gigabyte SQLite database (`kuma.db`)
  on slow SD cards, conditional `DELETE FROM heartbeat WHERE ...` causes
  heavy row-by-row B-tree/index updates and massive `.db-wal` growth. Instead,
  stash retained rows in `/dev/shm`, run an unconditional
  `DELETE FROM heartbeat` (`O(1)` table truncation), and compact in RAM via
  `VACUUM INTO '/dev/shm/kuma_compact.db'` without saturating SD card I/O.
