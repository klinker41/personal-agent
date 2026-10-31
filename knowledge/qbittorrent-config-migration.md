---
topic: qbittorrent-config-migration
category: knowledge
tags: [knowledge, qbittorrent-config-migration]
updated_at: 2026-10-24T06:00:43.267667+00:00
confidence: 0.95
---

# Knowledge: Qbittorrent-Config-Migration

- qBittorrent 5.3.0 runs a config migration (MigrationVersion 8 -> 12,
specifically Migration 11 `migrateWebUISettings`) on `qBittorrent.conf` that
moves all `WebUI\*` settings out of `[Preferences]` into a top-level `[WebUI]`
section (stripping the `WebUI\` prefix). Downgrading to 5.2.x without reverting
`qBittorrent.conf` causes 5.2.x to miss `WebUI\Password_PBKDF2` and fall back to
generating a random temporary password.
- When downgrading qBittorrent from 5.3.0 to 5.2.4, only `qBittorrent.conf`
needs to be restored from backup (while the container is stopped so in-memory
config isn't flushed on shutdown); `BT_backup/` (`.torrent` and `.fastresume`
files) and `qBittorrent-data.conf` (all-time stats) remain backward-compatible.
- After repeated failed login attempts against qBittorrent, the `qui` companion
service enters a 40-minute in-memory backoff timer (`backoffDuration: 2400000`
on IP ban detection); restarting `qui` or re-saving the instance settings clears
the backoff immediately.
