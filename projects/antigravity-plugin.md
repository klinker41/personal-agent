---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-09-25T00:32:18.199158+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Houses rules, skills, sidecars (`sidecar.json`), and lifecycle hooks at
`/workspace/antigravity-plugin`. Registered in `~/.gemini/config/plugins.json`,
tracks `main` (`git@github.com:klinker41/antigravity-plugin.git`), and vendors
skills via `vendor/agent-skills`.

## Invariants & Development Rules
- **Web Applications:** Bun runtime/package manager, Hono framework, minimal
  up-to-date dependencies (`rules/web-app-architecture.md`), and port 4401 via
  `https://prototype.klinker-cabin.computer` (`rules/web-service-port.md`).
- **Coding & Subagents:** Subagents for coding/tests use `Model: "flash"`
  (`rules/coding-subagent-model.md`). All rule/skill names must use kebab-case.
  Keep diffs minimal, update README on architectural changes, and enforce
  strict 80-character Markdown wrapping.
- **Git & Quality Gates:** Tests must pass non-verbosely before committing
  (`rules/non-verbose-tests.md`, `rules/tests-must-pass-before-commit.md`),
  zero secrets in diffs (`rules/no-secrets-in-commits.md`), mandatory
  `self-review-commit` loop (`Model: "pro"` reviewer), and user approval
  before `git push` (`rules/git-push.md`).

## Memory System (`sidecars/memory-daemon`)
- **Structure & Access:** Progressive disclosure store at `$MEMORY_DIRECTORY`
  (`profile.md`, `index.md`, `projects/`, `knowledge/`, and tiered
  `daily/`/`monthly/`/`yearly/` chronicles), queried via `lookup-memory` and
  updated via `save-memory`.
- **Pipeline & Lifecycle:** Turn-1 injection (`hooks/inject_memory.py`)
  triggers on `initialNumSteps == 0` and `invocationNum == 1`. Nightly local
  schedule: 00:00 Dreamer (persists step watermarks in `state.json`, checks
  topics via `memory_utils.get_existing_topics`, skips subagents and
  `rules/*.md`, purges ephemeral sessions); 00:30 tiered compaction; 01:00 Git
  sync with secret scrubbing.
- **Bridge & Discovery (`utils/memory_utils.py`):** `AgentApiBridge` manages
  `agentapi` subprocesses with auth retries. Strips caller session environment
  variables (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
  `ANTIGRAVITY_TRAJECTORY_ID`) to avoid project mismatch. Resolves projects in
  `~/.gemini/config/projects/` by name, UUID, or path (defaults to
  `personal-agent` `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc` or
  `$ANTIGRAVITY_PROJECT_ID`). Recovers `ANTIGRAVITY_LS_ADDRESS` and CSRF
  tokens (`window.__APP_CONFIG__.csrfToken`) from `cli.log` and runtime state.

## Sidecars & Integrations
- **Scheduled Tasks (`sidecar.json`):** Recurring `agentapi new-conversation`
  tasks with `--model` support: `model-updater` (Gemini defaults daily at
  15:00 UTC) and `submodule-updater` (`vendor/agent-skills` Mondays at
  15:00 UTC).
- **Slack Integration (`sidecars/slack-chat/`):** Connects Slack Socket Mode
  to `agentapi` via `AgentApiBridge`, mapping `thread_ts` to conversation IDs
  with `conversations_replies` backfill. Verified by `prep-slack-chat-sidecar`
  (`SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`); alerts sent via `notify-the-user`.
- **External AI Skills:** `ollama-chat` queries Gemma models hosted at
  `https://ollama.klinker-cabin.computer` using `$OLLAMA_API_KEY`.
