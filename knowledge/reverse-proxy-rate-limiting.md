---
topic: reverse-proxy-rate-limiting
category: knowledge
tags: [knowledge, reverse-proxy, openresty, nginx, caching, rate-limiting]
updated_at: 2026-10-30T14:36:10.703208+00:00
compacted_at: 2026-10-30T14:36:10.703208+00:00
confidence: 0.95
---

# Knowledge: Reverse Proxy Caching & Rate Limiting

- Extracting client IP directly from X-Forwarded-For headers without verifying
trusted proxy upstream hops enables trivial rate limiting bypass and IP
spoofing.

- When an edge reverse proxy (such as OpenResty) caches 404 Not Found responses
with long max-age headers, fixing backend asset routing will not resolve client
errors until the asset URL changes; forcing a content change in Vite produces a
new bundle hash that bypasses the cached 404.
