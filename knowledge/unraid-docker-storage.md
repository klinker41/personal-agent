---
topic: unraid-docker-storage
category: knowledge
tags: [knowledge, unraid-docker-storage]
updated_at: 2026-11-23T07:12:05.756756+00:00
confidence: 0.95
---

# Knowledge: Unraid-Docker-Storage

- Unmounted container paths and anonymous Docker volumes (such as CI agent
`~/.gradle`, `~/.m2`, and workspace directories) reside inside Unraid's
`/mnt/user/system/docker/docker.img` writable layers and persist until container
recreation; bind-mounting heavy build caches onto a cache-pool share prevents
`docker.img` utilization alerts while preserving caches across recreates.
