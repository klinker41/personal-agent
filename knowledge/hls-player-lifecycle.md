---
topic: hls-player-lifecycle
category: knowledge
tags: [knowledge, hls-player-lifecycle]
updated_at: 2026-09-16T00:00:51.034822+00:00
confidence: 0.95
---

# Knowledge: Hls-Player-Lifecycle

- Polling intervals checking for stream readiness can destroy in-flight Hls.js
instances if they re-trigger `startHlsPlayback()` before `MANIFEST_PARSED`
fires; guard player startup with an `isConnecting` flag.
- To avoid attaching players to premature or empty live playlists
(`LEVEL_EMPTY_ERROR`), verify that `live.m3u8` contains playable `#EXTINF`
segment tags before attaching Hls.js.
