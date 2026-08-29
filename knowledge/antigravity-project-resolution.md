---
topic: antigravity-project-resolution
category: knowledge
tags: [knowledge, antigravity-project-resolution]
updated_at: 2026-08-29T11:56:27.796179+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Project-Resolution

- Antigravity workspace projects are mapped to Project UUIDs in
`~/.gemini/config/projects/<UUID>.json`.
- Calling `agentapi new-conversation` requires the target workspace's Project
UUID (or inheriting `PROJECT_ID`/`AGY_PROJECT_ID`); passing a raw project name
string causes backend permission errors.
- Programmatic bridges can monitor execution of `agentapi new-conversation` by
watching the conversation's `transcript.jsonl` log file for completed
`PLANNER_RESPONSE` entries.
