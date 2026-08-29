---
topic: git-safe-directory
category: knowledge
tags: [knowledge, git-safe-directory]
updated_at: 2026-08-29T11:54:43.583094+00:00
confidence: 0.95
---

# Knowledge: Git-Safe-Directory

- Wildcard patterns can be used in git `safe.directory` configuration to trust
multiple repositories (e.g., `git config --global --add safe.directory
'/workspace/*'` or `'*'`).
- If `git config --global` fails due to bind-mounted file locks, the entry can
be added directly to `~/.gitconfig` under `[safe]` (`directory = /workspace/*`).
