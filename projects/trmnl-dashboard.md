---
topic: trmnl-dashboard
category: project
tags: [project, trmnl-dashboard]
updated_at: 2026-08-29T11:56:49.990104+00:00
confidence: 0.95
---

# Project: Trmnl-Dashboard

- Radarr WANTED metric is computed from `/api/v3/movie` filtering for `monitored
== True`, `hasFile == False`, and `isAvailable == True` to only count released
missing movies matching GetHomepage behavior.

- Supports pushing generated 800x480 PNG images directly to TRMNL via the
`TRMNL_WEBHOOK_URL` environment variable or `--webhook-url` CLI flag.
- Integrated async webhook publisher into both single-run and continuous
`--loop` execution modes.

- Lightweight Python service that queries homelab APIs and renders an 800x480
e-ink dashboard image via Pillow, replacing a heavier Homepage + Playwright
stack.
- Asynchronously queries Sonarr, Radarr, qBittorrent, Tdarr
(`StatisticsJSONDB`), Portainer (multi-environment), custom Home APIs (Energy,
Radon, Solar, unRAID), OpenWeatherMap, and Finnhub.
- Packaged in a minimal Alpine Docker container running on a 15-minute refresh
daemon loop (`--loop`) or single-shot execution via host cron.
- Repository hosted at git@github.com:klinker41/trmnl-dashboard.git.
