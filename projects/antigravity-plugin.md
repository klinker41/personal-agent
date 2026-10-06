---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-10-06T00:34:33.391250+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Repository at `/workspace/antigravity-plugin` (configured in
`~/.gemini/config/plugins.json`, tracking `main` from
`git@github.com:klinker41/antigravity-plugin.git`). Houses development
rules, skills, lifecycle hooks, background sidecars (`sidecar.json`), and
`vendor/agent-skills`.

## Development Rules & Invariants
- **Web Applications:** Bun runtime and package manager, Hono framework,
  minimal dependencies (`rules/web-app-architecture.md`), and port 4401
  routed via `https://prototype.klinker-cabin.computer`
  (`rules/web-service-port.md`).
- **Coding Standards:** Kebab-case rule/skill naming (`rules/rule-naming.md`,
  `rules/skill-naming.md`), 80-character Markdown wrap
  (`rules/markdown-formatting.md`), minimal diffs
  (`rules/simplify-changes.md`), README updates on architectural changes
  (`rules/readme-updates.md`), and `Model: "flash"` for coding/test subagents
  (`rules/coding-subagent-model.md`).
- **Git & Quality Gates:** Passing non-verbose tests
  (`rules/non-verbose-tests.md`, `rules/tests-must-pass-before-commit.md`)
  and zero secret leaks (`rules/no-secrets-in-commits.md`) required before
  commit. Mandatory iterative `self-review-commit` loop (`Model: "pro"`;
  `rules/self-review-before-commit.md`); user approval required before
  `git push` (`rules/git-push.md`).

## Memory System (`sidecars/memory-daemon`)
- **Structure & Access:** Progressive disclosure store at
  `$MEMORY_DIRECTORY` (`profile.md`, `index.md`, `projects/`, `knowledge/`,
  and tiered chronicles in `daily/`, `monthly/`, `yearly/`), accessed via
  `lookup-memory` and `save-memory`.
- **Pipeline & Lifecycle:** Turn-1 injection on `initialNumSteps == 0` and
  `invocationNum == 1` (`hooks/inject_memory.py`). Nightly daemon schedule:
  `00:00` Dreamer extraction (persists watermarks to `state.json`, checks
  topics via `memory_utils.get_existing_topics`, skips subagents and
  `rules/*.md`, purges ephemeral sessions), `00:30` tiered chronicle
  compaction, and `01:00` Git sync with secret scrubbing.

## Sidecars & Integrations
- **Agent API Bridge (`utils/memory_utils.py`):** `AgentApiBridge` executes
  `agentapi` subprocesses with auth retry, resolving projects in
  `~/.gemini/config/projects/` by name, UUID, or path (defaults to
  `personal-agent` UUID `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc` or
  `$ANTIGRAVITY_PROJECT_ID`). Strips caller session environment variables
  (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`, and
  `ANTIGRAVITY_TRAJECTORY_ID`) to avoid project mismatch; recovers
  `ANTIGRAVITY_LS_ADDRESS` and CSRF tokens
  (`window.__APP_CONFIG__.csrfToken`) from `cli.log` and runtime state.
- **Scheduled Maintenance (`sidecar.json`):** Recurring `agentapi` jobs
  (`new-conversation --model`): daily `model-updater` (Gemini defaults at
  15:00 UTC) and weekly `submodule-updater` (`vendor/agent-skills` Mondays at
  15:00 UTC).
- **Slack Integration (`sidecars/slack-chat/`):** Connects Slack Socket Mode
  to `agentapi` via `AgentApiBridge`, mapping `thread_ts` to conversation IDs
  with `conversations_replies` backfill. Verified by
  `prep-slack-chat-sidecar` (`SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`); alerts
  sent via `notify-the-user`.
- **External AI Skills:** `ollama-chat` queries Gemma models hosted at
  `https://ollama.klinker-cabin.computer` using `$OLLAMA_API_KEY`.
