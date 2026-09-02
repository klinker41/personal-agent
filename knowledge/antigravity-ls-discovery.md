---
topic: antigravity-ls-discovery
category: knowledge
tags: [knowledge, antigravity-ls-discovery]
updated_at: 2026-09-02T00:00:28.555592+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Ls-Discovery

- The Antigravity Language Server assigns dynamic TCP ports across sessions;
static `ANTIGRAVITY_LS_ADDRESS` values lead to `connection refused` RPC errors
in `agentapi` if the server restarts.
- Dynamic recovery involves validating address liveness with a TCP socket probe
and falling back to parsing `cli.log` or runtime state files in
`~/.gemini/antigravity/sidecar_data/`.

- The `agy` CLI supports the `--hub-port <port>` flag in `--remote-control` mode
to bind the hub/language server to a deterministic port rather than defaulting
to random port `0`, eliminating the need to scrape stdout or logs for dynamic
port discovery.
