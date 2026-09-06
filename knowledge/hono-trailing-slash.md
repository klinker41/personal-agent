---
topic: hono-trailing-slash
category: knowledge
tags: [knowledge, hono-trailing-slash]
updated_at: 2026-09-06T00:00:17.124351+00:00
confidence: 0.95
---

# Knowledge: Hono-Trailing-Slash

- Use Hono's official trimTrailingSlash() middleware rather than monkey-patching
app.fetch, which risks prematurely consuming or corrupting the incoming request
body stream.
