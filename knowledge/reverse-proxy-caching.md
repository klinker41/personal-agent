---
topic: reverse-proxy-caching
category: knowledge
tags: [knowledge, reverse-proxy-caching]
updated_at: 2026-09-06T00:00:17.124236+00:00
confidence: 0.95
---

# Knowledge: Reverse-Proxy-Caching

- When an edge reverse proxy (such as OpenResty) caches 404 Not Found responses
with long max-age headers, fixing backend asset routing will not resolve client
errors until the asset URL changes; forcing a content change in Vite produces a
new bundle hash that bypasses the cached 404.
