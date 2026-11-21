---
topic: radarr-api-metrics
category: knowledge
tags: [knowledge, radarr-api-metrics]
updated_at: 2026-11-21T07:02:17.753483+00:00
confidence: 0.95
---

# Knowledge: Radarr-Api-Metrics

- Radarr's `/api/v3/wanted/missing` endpoint includes all unreleased monitored
movies without files. To count only missing movies that have passed their
release window, filter `/api/v3/movie` for `monitored == True`, `hasFile ==
False`, and `isAvailable == True`.

- Radarr and Sonarr share compatible Arr v3 queue endpoints and status payloads,
allowing a single shared provider implementation for queue inspection,
qBittorrent tracker classification, and safe queue item removal.
