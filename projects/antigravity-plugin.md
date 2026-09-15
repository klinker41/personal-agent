---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-09-15T00:32:24.259452+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Antigravity plugin packaging rules, skills, sidecars (`sidecar.json`), and
lifecycle hooks at `/workspace/antigravity-plugin`. Registered in
`~/.gemini/config/plugins.json`, tracking `main` at
`git@github.com:klinker41/antigravity-plugin.git`, and vendoring skills via
`vendor/agent-skills`.

## Invariants & Rules
- **Web Applications:** Bun runtime/package manager, Hono framework, minimal
  up-to-date dependencies (`rules/web-app-architecture.md`), and port 4401 via
  `https://prototype.klinker-cabin.computer` (`rules/web-service-port.md`).
- **Git & Quality:** Passing non-verbose tests, zero secrets in diffs, mandatory
  `self-review-commit` loop (with `Model="pro"` reviewer subagent), and explicit
  user approval before `git push` (`rules/self-review-before-commit.md`,
  `rules/no-secrets-in-commits.md`, `rules/git-push.md`).
- **Standards & Subagents:** Use `Model: "flash"` for coding/testing subagents
  (`rules/coding-subagent-model.md`). Enforce kebab-case names, minimal diffs,
  no duplication, README updates, and strict 80-character Markdown wrapping.

## Memory Architecture & Daemon (`sidecars/memory-daemon`)
- **Structure:** Progressive disclosure store at `$MEMORY_DIRECTORY`
  (`profile.md`, `index.md`, `projects/`, `knowledge/`, and tiered
  `daily/`/`monthly/`/`yearly/` chronicles).
- **Hybrid Pipeline:** `hooks/inject_memory.py` injects memory on turn 1
  (`initialNumSteps == 0`, `invocationNum == 1`, inspecting transcripts).
  Deterministic Python scrubs secrets and syncs Git; `agentapi` handles LLM
  extraction, synthesis, and compaction.
- **Nightly Maintenance:** Dreamer at 00:00 (tracks step watermarks in
  `state.json`, checks `memory_utils.get_existing_topics` to prevent duplicates,
  skips subagents and `rules/*.md`, purges ephemeral sessions); tiered
  compaction at 00:30; Git sync at 01:00 local time.

## Shared Utilities (`utils/memory_utils.py`)
- **Dynamic LS Discovery:** Probes runtime state and active `cli.log` for
  live `ANTIGRAVITY_LS_ADDRESS` and `ANTIGRAVITY_CSRF_TOKEN` when env vars are
  stale.
- **AgentApiBridge:** Subprocess runner for `agentapi`. Strips caller env vars
  (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
  `ANTIGRAVITY_TRAJECTORY_ID`) to avoid project mismatch, extracts web hub CSRF
  tokens (`window.__APP_CONFIG__.csrfToken`), and retries auth failures.
- **Project Resolution:** Resolves projects against `~/.gemini/config/projects/`
  by name, UUID, or path, falling back to `personal-agent`
  (`6b1d3dc5-a020-4710-94f5-79b34fc1b9fc`) or `$ANTIGRAVITY_PROJECT_ID` /
  `$PROJECT_ID` / `$AGY_PROJECT_ID`.

## Sidecars & Skills
- **Scheduled Sidecars (`sidecar.json`):** Recurring `agentapi new-conversation`
  tasks (supporting `--model` overrides). Runs `model-updater` (Gemini defaults
  daily at 15:00 UTC) and `submodule-updater` (`vendor/agent-skills` Mondays at
  15:00 UTC).
- **Slack Integration:** `sidecars/slack-chat/` bridges Slack Socket Mode to
  `agentapi` via `AgentApiBridge`, mapping `thread_ts` to conversation IDs with
  `conversations_replies` backfill. Validated by `prep-slack-chat-sidecar`
  (`SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`). `notify-the-user` sends webhook
  alerts.
- **Chat & Memory Skills:** `lookup-memory` and `save-memory` manage
  `$MEMORY_DIRECTORY`; `ollama-chat` queries Gemma models at
  `https://ollama.klinker-cabin.computer` via `$OLLAMA_API_KEY`.
