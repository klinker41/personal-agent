---
topic: jellyfin-memory-tuning
category: knowledge
tags: [knowledge, jellyfin-memory-tuning]
updated_at: 2026-11-22T07:04:43.149385+00:00
confidence: 0.95
---

# Knowledge: Jellyfin-Memory-Tuning

- Jellyfin defaults to
<ParallelImageEncodingLimit>0</ParallelImageEncodingLimit> in system.xml
(unbounded concurrency across CPU threads); decoding ultra-high-resolution JPEGs
into uncompressed 32-bit RGBA bitmaps (~4 bytes/pixel, e.g. ~2 GB RAM per 511 MP
image) during .webp thumbnail generation can rapidly exhaust RAM and freeze
swapless hosts like Unraid.
- Cap <ParallelImageEncodingLimit> (e.g. to 2) in Jellyfin's config/system.xml
alongside a Docker memory limit so heavy library scans stay bounded or trigger a
contained cgroup OOM rather than hanging the host OS.
- Use docker update --memory=<limit> <container> to apply cgroup memory caps
live to running containers without restarting them.

- Following the 96GB RAM upgrade on unraid-nas, the 8g Docker container memory
limit on Jellyfin was removed while keeping
<ParallelImageEncodingLimit>2</ParallelImageEncodingLimit> in
/mnt/user/docker/jellyfin/config/config/system.xml.
