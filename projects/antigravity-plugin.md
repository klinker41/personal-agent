---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-09-13T00:31:26.913904+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Antigravity plugin packaging rules, skills, sidecars (`sidecar.json`), and
lifecycle hooks at `/workspace/antigravity-plugin`. Registered in
`~/.gemini/config/plugins.json`, tracking `main` at
`git@github.com:klinker41/antigravity-plugin.git`, vendoring skills via
`vendor/agent-skills`.

## Invariants & Rules
- **Web Applications:** Bun runtime/package manager, Hono framework, latest
  minimal dependencies (`rules/web-app-architecture.md`), and port 4401 via
  `https://prototype.klinker-cabin.computer` (`rules/web-service-port.md`).
- **Git & Commits:** Mandatory `self-review-commit` pre-commit loop (with
  `Model="pro"` reviewer subagent), zero secrets in diffs, and explicit
  user approval before `git push` (`rules/self-review-before-commit.md`,
  `rules/no-secrets-in-commits.md`, `rules/git-push.md`).
- **Engineering Standards:** Minimal diffs, no code duplication, kebab-case
  rule/skill names, non-verbose tests, updated README, and strict 80-character
  Markdown line wrapping (`rules/*.md`).

## Memory Architecture & Daemon (`sidecars/memory-daemon`)
- **Structure:** Progressive disclosure store at `$MEMORY_DIRECTORY`
  (`profile.md`, `index.md`, `projects/`, `knowledge/`, and tiered
  `daily/`/`monthly/`/`yearly/` chronicles).
- **Hybrid Pipeline & Hooks:** Deterministic Python scrubs secrets, syncs
  git, and performs turn-1 memory injection via `hooks/inject_memory.py`
  (`initialNumSteps == 0`, `invocationNum == 1`, checking transcripts).
  `agentapi` handles LLM extraction, synthesis, and compaction.
- **Nightly Maintenance:** Dreamer at 00:00 (tracks step watermarks in
  `state.json`, checks `memory_utils.get_existing_topics` to avoid duplicate
  notes, skips subagents and `rules/*.md`, purges ephemeral sessions); tiered
  compaction at 00:30; Git sync at 01:00 local time.

## Shared Utilities (`utils/memory_utils.py`)
- **Dynamic LS Discovery:** Probes runtime state and active `cli.log` for
  live `ANTIGRAVITY_LS_ADDRESS` and `ANTIGRAVITY_CSRF_TOKEN` when env vars are
  stale.
- **AgentApiBridge:** Subprocess runner for `agentapi`. Clears caller env
  vars (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
  `ANTIGRAVITY_TRAJECTORY_ID`) to avoid project mismatch, extracts web hub
  CSRF tokens (`window.__APP_CONFIG__.csrfToken`), and retries auth failures.
- **Project Resolution:** Resolves projects against
  `~/.gemini/config/projects/` by name, UUID, or path, falling back to
  `personal-agent` (`6b1d3dc5-a020-4710-94f5-79b34fc1b9fc`) or
  `$ANTIGRAVITY_PROJECT_ID` / `$PROJECT_ID` / `$AGY_PROJECT_ID`.

## Sidecars & Skills
- **Scheduled Sidecars (`sidecar.json`):** Configure recurring tasks via
  `agentapi new-conversation` in `args` (supporting `--model` overrides).
  Runs `model-updater` (Gemini defaults daily at 15:00 UTC) and
  `submodule-updater` (`vendor/agent-skills` Mondays at 15:00 UTC).
- **Slack Integration:** `sidecars/slack-chat/` bridges Slack Socket Mode to
  `agentapi` via `AgentApiBridge`, mapping `thread_ts` to conversation IDs
  with `conversations_replies` backfill. Config validated by
  `skills/prep-slack-chat-sidecar` (`SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`).
  `notify-the-user` sends Slack webhook alerts.
- **Chat & Memory Skills:** `ollama-chat` (queries Gemma models at
  `https://ollama.klinker-cabin.computer` via `$OLLAMA_API_KEY`);
  `lookup-memory` and `save-memory` (on-demand retrieval and persistence in
  `$MEMORY_DIRECTORY`).
