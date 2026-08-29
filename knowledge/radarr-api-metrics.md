---
topic: radarr-api-metrics
category: knowledge
tags: [knowledge, radarr-api-metrics]
updated_at: 2026-08-29T11:55:15.959985+00:00
confidence: 0.95
---

# Knowledge: Radarr-Api-Metrics

- Radarr's `/api/v3/wanted/missing` endpoint includes all unreleased monitored
movies without files. To count only missing movies that have passed their
release window, filter `/api/v3/movie` for `monitored == True`, `hasFile ==
False`, and `isAvailable == True`.
