---
topic: sonarr-docker-mounts
category: knowledge
tags: [knowledge, sonarr-docker-mounts]
updated_at: 2026-10-23T06:03:18.926425+00:00
confidence: 0.95
---

# Knowledge: Sonarr-Docker-Mounts

- Mounting download and media directories as separate Docker bind mounts (e.g.,
`/data/completed` and `/data/TV`) prevents atomic hardlinks and forces Sonarr
to perform full file copies on import, which can trigger transient I/O and
SQLite lock contention that stalls HTTP health checks.
