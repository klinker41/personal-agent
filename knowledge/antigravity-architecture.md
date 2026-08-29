---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-08-29T11:54:47.194212+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- Antigravity Web UI communicates via local RPC/WebSockets to the compiled local
agy daemon, which directly executes tools and sends outbound HTTPS requests to
Gemini API servers rather than making model calls directly from the browser.
