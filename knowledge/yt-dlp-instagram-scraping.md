---
topic: yt-dlp-instagram-scraping
category: knowledge
tags: [knowledge, yt-dlp-instagram-scraping]
updated_at: 2026-11-08T07:02:58.812801+00:00
confidence: 0.95
---

# Knowledge: Yt-Dlp-Instagram-Scraping

- To minimize Instagram automated-behavior flags when polling a profile's
`/reels/` feed with `yt-dlp`, combine `--download-archive` and
`--playlist-end 3` with human-like pacing (`--sleep-requests 3`,
`--sleep-interval 5 --max-sleep-interval 15`), a standard desktop browser
`User-Agent`, schedule jitter, and a working writable copy of `cookies.txt` from
an aged account on a residential IP without concurrent mobile app sessions.
