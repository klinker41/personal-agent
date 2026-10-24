---
topic: antigravity-web-client
category: knowledge
tags: [knowledge, antigravity-web-client]
updated_at: 2026-10-24T06:00:49.403280+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Web-Client

- Antigravity web UI stores the active project selection in browser localStorage
per client, so accessing the same instance from a new device requires selecting
the project before scoped conversations appear.

- When spoofing a desktop `navigator.userAgent` on foldable mobile devices to
force the side-by-side split-pane layout, Antigravity's Lexical prompt editor
disables `mobileLayout` and treats unmodified `Enter` key presses as message
submission rather than `INSERT_LINE_BREAK_COMMAND` ; overriding
`shiftKey = true` during capture-phase `keydown` restores mobile newline
behavior.
