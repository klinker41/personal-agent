---
topic: tdarr-version-parity
category: knowledge
tags: [knowledge, tdarr-version-parity]
updated_at: 2026-11-27T07:08:21.497586+00:00
confidence: 0.95
---

# Knowledge: Tdarr-Version-Parity

- Tdarr enforces strict exact version parity between `tdarr-server` and
`tdarr-node`; if one container updates to `:latest` while the other remains on
an older image during a partial stack update, `tdarr-node` fails startup every
10 seconds with `Node version X does not match Server version Y` until the
lagging container is recreated.
