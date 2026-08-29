---
topic: antigravity-ls-discovery
category: knowledge
tags: [knowledge, antigravity-ls-discovery]
updated_at: 2026-08-29T11:53:21.103643+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Ls-Discovery

- The Antigravity Language Server assigns dynamic TCP ports across sessions;
static `ANTIGRAVITY_LS_ADDRESS` values lead to `connection refused` RPC errors
in `agentapi` if the server restarts.
- Dynamic recovery involves validating address liveness with a TCP socket probe
and falling back to parsing `cli.log` or runtime state files in
`~/.gemini/antigravity/sidecar_data/`.
