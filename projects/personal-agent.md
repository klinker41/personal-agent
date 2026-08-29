---
topic: personal-agent
category: project
tags: [project, personal-agent]
updated_at: 2026-08-29T11:57:28.467102+00:00
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
