---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-10-01T00:34:29.273962+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Repository at `/workspace/antigravity-plugin` (registered in
`~/.gemini/config/plugins.json`, tracking `main` from
`git@github.com:klinker41/antigravity-plugin.git`). Contains development rules,
skills, lifecycle hooks, sidecars (`sidecar.json`), and `vendor/agent-skills`.

## Development Rules & Invariants
- **Web Applications:** Bun runtime/package manager, Hono framework, minimal
  dependencies (`rules/web-app-architecture.md`), and port 4401 routed via
  `https://prototype.klinker-cabin.computer` (`rules/web-service-port.md`).
- **Coding Standards:** Kebab-case naming for rules and skills
  (`rules/rule-naming.md`, `rules/skill-naming.md`), `Model: "flash"` for coding
  and test subagents (`rules/coding-subagent-model.md`), minimal diffs
  (`rules/simplify-changes.md`), README updates on architectural changes
  (`rules/readme-updates.md`), and 80-character line wrap on Markdown files
  (`rules/markdown-formatting.md`).
- **Git & Quality Gates:** Tests must pass non-verbosely before committing
  (`rules/non-verbose-tests.md`, `rules/tests-must-pass-before-commit.md`), zero
  secrets in diffs (`rules/no-secrets-in-commits.md`), mandatory
  `self-review-commit` loop (`Model: "pro"` reviewer;
  `rules/self-review-before-commit.md`), and user approval required before
  `git push` (`rules/git-push.md`).

## Memory System (`sidecars/memory-daemon`)
- **Structure & Access:** Progressive disclosure store at `$MEMORY_DIRECTORY`
  (`profile.md`, `index.md`, `projects/`, `knowledge/`, and tiered chronicles in
  `daily/`, `monthly/`, `yearly/`). Read via `lookup-memory`, written via
  `save-memory`.
- **Pipeline & Lifecycle:** Turn-1 injection (`hooks/inject_memory.py`) triggers
  on `initialNumSteps == 0` and `invocationNum == 1`. Automated operations:
  - `00:00`: Dreamer extracts updates (persists watermarks in `state.json`,
    checks topics via `memory_utils.get_existing_topics`, skips subagents and
    `rules/*.md`, purges ephemeral sessions).
  - `00:30`: Tiered chronicle compaction.
  - `01:00`: Git sync with secret scrubbing.
- **Bridge & Discovery (`utils/memory_utils.py`):** `AgentApiBridge` executes
  `agentapi` subprocesses with auth retry. Strips caller session environment
  variables (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`, and
  `ANTIGRAVITY_TRAJECTORY_ID`) to prevent project mismatch. Resolves projects in
  `~/.gemini/config/projects/` by name, UUID, or path (defaults to
  `personal-agent` UUID `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc` or
  `$ANTIGRAVITY_PROJECT_ID`). Recovers `ANTIGRAVITY_LS_ADDRESS` and CSRF tokens
  (`window.__APP_CONFIG__.csrfToken`) from `cli.log` and runtime state.

## Sidecars & Integrations
- **Scheduled Tasks (`sidecar.json`):** Recurring jobs using
  `agentapi new-conversation` with `--model`: `model-updater` (Gemini defaults
  daily at 15:00 UTC) and `submodule-updater` (`vendor/agent-skills` Mondays
  at 15:00 UTC).
- **Slack Integration (`sidecars/slack-chat/`):** Connects Slack Socket Mode to
  `agentapi` via `AgentApiBridge`, mapping `thread_ts` to conversation IDs with
  `conversations_replies` backfill. Verified by `prep-slack-chat-sidecar`
  (`SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`); alerts sent via `notify-the-user`.
- **External AI Skills:** `ollama-chat` queries Gemma models hosted at
  `https://ollama.klinker-cabin.computer` using `$OLLAMA_API_KEY`.
