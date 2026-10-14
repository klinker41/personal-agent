---
topic: dockerfile-cmd-syntax
category: knowledge
tags: [knowledge, dockerfile-cmd-syntax]
updated_at: 2026-10-14T00:03:29.536087+00:00
confidence: 0.95
---

# Knowledge: Dockerfile-Cmd-Syntax

- When an exec-form JSON array in a Dockerfile `CMD` instruction is malformed or
unclosed, Docker falls back to shell form (`/bin/sh -c '...'`), which causes
runtime syntax errors such as
`/bin/sh: 1: Syntax error: Unterminated quoted string` rather than a build-time
failure.
