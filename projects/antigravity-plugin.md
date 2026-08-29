---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-08-28T21:00:00Z
confidence: 1.0
---

# Project Context: Antigravity Plugin

Custom plugin bundle for Google Antigravity packaging rules, skills, sidecars,
and lifecycle hooks.

## Key Architectural Invariants
- **Web Service Port:** All web services must run on port 4401.
- **Sidecar Lifecycle:** Background services run as daemons managed via
  `sidecar.json`.
- **Memory Integration:** Powered by the `memory-daemon` sidecar,
  `lookup-memory` and `save-memory` skills, and PreInvocation hooks.
- **Git Push Policy:** Always use `git commit` after self-review; never run
  `git push` automatically on this repo without explicit confirmation.
- **Line Wrapping:** Strict 80-character maximum per line in `.md` files.
