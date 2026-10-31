---
topic: uptime-kuma-diagnostics
category: knowledge
tags: [knowledge, uptime-kuma-diagnostics, uptime-kuma, komodo, pi-studio]
updated_at: 2026-10-30T14:35:00+00:00
compacted_at: 2026-10-30T14:35:00+00:00
confidence: 0.95
---

# Knowledge: Uptime-Kuma-Diagnostics

- Because Uptime Kuma (`uptime.klinker.computer`) runs on `pi-studio` in Boulder
and probes `.klinker-cabin.computer` services in Estes across an inter-site WAN
/ `gluetun-home` WireGuard link, transient inter-site connectivity drops cause
48,000ms probe timeout alerts for Cabin-hosted services (`unraid-nas`,
`unraid-seedbox`, `unraid-aibox`) even when local containers and ingress
endpoints are healthy.
- Uptime Kuma API keys (`UPTIME_KUMA_API_KEY`) only authenticate the read-only
Prometheus `/metrics` endpoint; mutating monitors or status pages
(`deleteMonitor`, `saveStatusPage`) requires authenticating via Uptime Kuma's
Socket.IO WebSocket with an admin username and password.
- When `kuma_get_monitors` returns `HTTP 502 Bad Gateway` rather than `HTTP 0`
(timeout), the `pi-studio` host and `pi-studio-reverse-proxy` (`openresty`) are
online and reachable over WAN, isolating the failure to the upstream
`uptime-kuma` container (`pi-studio-dashboard` stack) restarting or
initializing.
- When pruning millions of `heartbeat` rows or compacting a multi-gigabyte
Uptime Kuma SQLite database (`kuma.db`) on slow SD card storage, a conditional
`DELETE FROM heartbeat WHERE ...` causes heavy row-by-row B-tree and index
updates plus massive `.db-wal` growth; stashing retained rows in `/dev/shm`
followed by an unconditional `DELETE FROM heartbeat` (`O(1)` table truncation)
and `VACUUM INTO '/dev/shm/kuma_compact.db'` performs compaction in RAM without
saturating SD card I/O.
- On `unraid-nas`, `semaphore-boulder` (`Ansible Boulder`) uses
`network_mode: service:gluetun-home` (in the Komodo `management` stack) and
`rsync-backup` uses `network_mode: service:gluetun-home-backup` (in the Komodo
`files` stack). When `gluetun-home` or `gluetun-home-backup` is recreated (e.g.,
during the 3:00 AM container auto-update window), dependent containers remain
pinned to the destroyed container ID, causing `Ansible Boulder` to fail with
HTTP 502; `komodo_restart_container` fails with `No such container: <old_id>`,
so the dependent container must be recreated via
```bash
docker exec -w /mnt/user/docker/komodo/periphery/stacks/nas-management/stacks/unraid-nas/management \
  komodo-periphery docker compose -p management --env-file .env up -d --force-recreate semaphore-boulder
```
