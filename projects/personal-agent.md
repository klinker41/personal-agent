---
topic: personal-agent
category: project
tags: [project, personal-agent]
updated_at: 2026-08-30T11:29:40.438718+00:00
confidence: 0.95
---

# Project: Personal-Agent

- Repository location is configured via the `MEMORY_DIRECTORY` environment
  variable.
- Uses a tiered hierarchical compaction model: daily logs
  (`daily/YYYY-MM-DD.md`, 30-day retention) compact into monthly chronicles
  (`monthly/YYYY-MM.md`, 12-month retention), which compact into yearly archives
  (`yearly/YYYY.md`).
- Directory structure includes `profile.md`, `index.md`, `knowledge/`,
  `projects/`, `daily/`, `monthly/`, and `yearly/`.

- Git repository initialized with remote origin
  git@github.com:klinker41/personal-agent.git and default branch main.

- Memory daemon sidecar executes scheduled phases: Nightly Dream Consolidation
  at 02:00 UTC, Tiered Compaction at 02:30 UTC, and Git Synchronization at 03:00
  UTC using `agy agentapi`, logging to:
  `~/.gemini/antigravity/sidecar_data/antigravity-plugin/memory-daemon/logs/worker.log`.
