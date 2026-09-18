---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-09-18T00:31:44.492904+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Packages rules, skills, sidecars (`sidecar.json`), and lifecycle hooks at
`/workspace/antigravity-plugin`. It is registered in
`~/.gemini/config/plugins.json`, tracks `main` at
`git@github.com:klinker41/antigravity-plugin.git`, and vendors skills via
`vendor/agent-skills`.

## Invariants & Development Rules
- **Web Applications:** Bun runtime and package manager, Hono framework,
  minimal up-to-date dependencies (`rules/web-app-architecture.md`), and port
  4401 via `https://prototype.klinker-cabin.computer`
  (`rules/web-service-port.md`).
- **Git & Review:** Passing non-verbose tests (`rules/non-verbose-tests.md`,
  `rules/tests-must-pass-before-commit.md`), zero secrets in diffs
  (`rules/no-secrets-in-commits.md`), mandatory `self-review-commit` loop
  (`Model: "pro"` reviewer), and user approval before `git push`
  (`rules/git-push.md`).
- **Coding Standards:** Subagents for coding/tests use `Model: "flash"`
  (`rules/coding-subagent-model.md`). Enforce kebab-case for rules/skills,
  minimal non-duplicative diffs, README updates, and strict 80-character
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
  `state.json`, checks topics via `memory_utils.get_existing_topics`, skips
  subagents and `rules/*.md`, purges ephemeral sessions); tiered compaction
  at 00:30; Git sync at 01:00 local time.
- **Shared Utilities (`utils/memory_utils.py`):**
  - *AgentApiBridge:* Subprocess runner for `agentapi`. Strips caller env vars
    (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
    `ANTIGRAVITY_TRAJECTORY_ID`) to avoid project mismatch; extracts web hub
    CSRF token (`window.__APP_CONFIG__.csrfToken`); retries on auth failure.
  - *Runtime & Project Resolution:* Matches `~/.gemini/config/projects/` by
    name, UUID, or path (defaults to `personal-agent`
    `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc` or `$ANTIGRAVITY_PROJECT_ID`);
    recovers stale `ANTIGRAVITY_LS_ADDRESS` and CSRF tokens from active
    `cli.log` and runtime state.

## Sidecars & Integrations
- **Scheduled Tasks (`sidecar.json`):** Recurring `agentapi new-conversation`
  tasks with `--model` support. Runs `model-updater` (Gemini defaults daily at
  15:00 UTC) and `submodule-updater` (`vendor/agent-skills` Mondays at
  15:00 UTC).
- **Slack Integration:** `sidecars/slack-chat/` connects Slack Socket Mode
  to `agentapi` via `AgentApiBridge`, mapping `thread_ts` to conversation IDs
  with `conversations_replies` backfill. Verified by `prep-slack-chat-sidecar`
  (`SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`); `notify-the-user` sends alerts.
- **External AI Skills:** `ollama-chat` queries Gemma models hosted at
  `https://ollama.klinker-cabin.computer` using `$OLLAMA_API_KEY`.
