---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-09-17T00:31:46.210755+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Packages rules, skills, sidecars (`sidecar.json`), and lifecycle hooks at
`/workspace/antigravity-plugin`. Registered in `~/.gemini/config/plugins.json`,
tracks `main` at `git@github.com:klinker41/antigravity-plugin.git`, and vendors
skills via `vendor/agent-skills`.

## Invariants & Development Rules
- **Web Applications:** Use Bun runtime/package manager, Hono framework,
  minimal up-to-date dependencies (`rules/web-app-architecture.md`), and port
  4401 via `https://prototype.klinker-cabin.computer`
  (`rules/web-service-port.md`).
- **Git & Quality:** Passing non-verbose tests, zero secrets in diffs,
  mandatory `self-review-commit` loop (`Model="pro"` reviewer), and explicit
  user approval before `git push` (`rules/self-review-before-commit.md`,
  `rules/no-secrets-in-commits.md`, `rules/git-push.md`).
- **Coding Standards:** Use `Model: "flash"` for coding/testing subagents
  (`rules/coding-subagent-model.md`). Enforce kebab-case rules and skills,
  minimal diffs, no duplication, README updates, and strict 80-character
  Markdown wrapping.

## Memory System (`sidecars/memory-daemon`)
- **Structure & Access:** Progressive disclosure store at `$MEMORY_DIRECTORY`
  (`profile.md`, `index.md`, `projects/`, `knowledge/`, and tiered
  `daily/`/`monthly/`/`yearly/` chronicles), managed via `lookup-memory` and
  `save-memory` skills.
- **Pipeline & Lifecycle:** Turn-1 injection via `hooks/inject_memory.py`
  (`initialNumSteps == 0`, `invocationNum == 1`, inspecting transcripts).
  Deterministic Python scrubs secrets and syncs Git; `agentapi` handles LLM
  extraction, synthesis, and compaction.
- **Nightly Maintenance:** Dreamer at 00:00 (tracks step watermarks in
  `state.json`, checks existing topics via `memory_utils.get_existing_topics`,
  skips subagents and `rules/*.md`, purges ephemeral sessions); tiered
  compaction at 00:30; Git sync at 01:00 local time.
- **Shared Utilities (`utils/memory_utils.py`):**
  - *Runtime Discovery:* Probes active `cli.log` and runtime state for live
    `ANTIGRAVITY_LS_ADDRESS` and `ANTIGRAVITY_CSRF_TOKEN` when env vars are
    stale.
  - *AgentApiBridge:* Subprocess runner for `agentapi`. Strips caller env
    vars (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
    `ANTIGRAVITY_TRAJECTORY_ID`) to avoid project mismatch; extracts web hub
    CSRF tokens (`window.__APP_CONFIG__.csrfToken`); retries on auth failure.
  - *Project Resolution:* Matches `~/.gemini/config/projects/` by name, UUID,
    or path; falls back to `personal-agent`
    (`6b1d3dc5-a020-4710-94f5-79b34fc1b9fc`) or project env vars
    (`$ANTIGRAVITY_PROJECT_ID` / `$PROJECT_ID` / `$AGY_PROJECT_ID`).

## Sidecars & Integrations
- **Scheduled Tasks (`sidecar.json`):** Recurring `agentapi new-conversation`
  tasks (supports `--model` overrides). Runs `model-updater` (Gemini
  defaults daily at 15:00 UTC) and `submodule-updater`
  (`vendor/agent-skills` Mondays at 15:00 UTC).
- **Slack Integration:** `sidecars/slack-chat/` connects Slack Socket Mode
  to `agentapi` via `AgentApiBridge`, mapping `thread_ts` to conversation IDs
  with `conversations_replies` backfill. Verified by
  `prep-slack-chat-sidecar` (`SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`);
  `notify-the-user` dispatches webhook alerts.
- **External AI Skills:** `ollama-chat` queries Gemma models hosted at
  `https://ollama.klinker-cabin.computer` using `$OLLAMA_API_KEY`.
