---
topic: unraid-docker-storage
category: knowledge
tags: [knowledge, unraid-docker-storage]
updated_at: 2026-11-27T07:04:28.746921+00:00
confidence: 0.95
---

# Knowledge: Unraid-Docker-Storage

- Unmounted container paths and anonymous Docker volumes (such as CI agent
`~/.gradle`, `~/.m2`, and workspace directories) reside inside Unraid's
`/mnt/user/system/docker/docker.img` writable layers and persist until container
recreation; bind-mounting heavy build caches onto a cache-pool share prevents
`docker.img` utilization alerts while preserving caches across recreates.

- On `unraid-nas`, `/mnt/user/system/docker/docker.img` is 150 GiB Btrfs with
~38 GiB (~26%) steady-state utilization; unmounted container paths like `/tmp`
write directly into `docker.img` writable layers and can trigger Unraid's >70%
disk utilization warning during heavy temporary I/O unless bind-mounted to an
Unraid host share.
