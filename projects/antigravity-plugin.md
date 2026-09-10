---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-09-10T00:31:54.893201+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Bundle packaging rules, skills, sidecars (declared in `sidecar.json`), and
lifecycle hooks for Google Antigravity. Located at
`/workspace/antigravity-plugin`, registered in `~/.gemini/config/plugins.json`,
tracking remote `main` at `git@github.com:klinker41/antigravity-plugin.git`,
and vendoring skills via the `vendor/agent-skills` git submodule.

## Key Invariants & Rules
- **Web Applications & Services:** Use Bun and Hono, minimize dependencies to
  latest versions (`rules/web-app-architecture.md`), and run on port 4401 via
  `https://prototype.klinker-cabin.computer` (`rules/web-service-port.md`).
- **Git & Commit Safeguards:** Run iterative `self-review-commit`, reject
  secrets in diffs, and require explicit confirmation for `git push`
  (`rules/git-push.md`, `rules/no-secrets-in-commits.md`,
  `rules/self-review-before-commit.md`).
- **Code & Markdown Standards:** Minimize diffs, eliminate extraneous edits,
  deduplicate shared code (`rules/simplify-changes.md`), and enforce strict
  80-character line wrapping in Markdown (`rules/markdown-formatting.md`).

## Memory Architecture & Daemon (`sidecars/memory-daemon`)
- **Structure & Storage:** Progressive disclosure repository at
  `$MEMORY_DIRECTORY` (`profile.md`, `index.md`, `projects/`, `knowledge/`, and
  tiered `daily/`, `monthly/`, `yearly/` chronicles).
- **Hybrid Processing & Hooks:** Deterministic Python handles secret scrubbing,
  Git sync, and turn-1 memory injection via `hooks/inject_memory.py`
  (`initialNumSteps == 0`, `invocationNum == 1`, checking transcripts to avoid
  redundancy). `agentapi` handles LLM extraction, synthesis, and compaction.
- **Maintenance & Extraction:** Nightly maintenance runs dreaming at 00:00,
  tiered compaction at 00:30, and Git sync at 01:00 local time. The dreamer
  tracks step watermarks in `state.json` for incremental extraction, skips
  subagents and rules covered in `rules/*.md`, queries topic catalogs via
  `memory_utils.get_existing_topics` to prevent duplicates, and purges
  ephemeral internal conversations.

## Shared Utilities (`utils/memory_utils.py`)
- **Dynamic LS Discovery:** Probes runtime state and active `cli.log` for live
  `ANTIGRAVITY_LS_ADDRESS` and `ANTIGRAVITY_CSRF_TOKEN` when env vars are stale.
- **AgentApiBridge:** Subprocess runner for `agentapi`. Strips caller metadata
  (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
  `ANTIGRAVITY_TRAJECTORY_ID`) to avoid project mismatch errors, fetches web
  hub CSRF tokens (`window.__APP_CONFIG__.csrfToken`), and retries auth
  failures.
- **Project ID Resolution:** Resolves project IDs against
  `~/.gemini/config/projects/` by name, UUID, or path, falling back to
  `DEFAULT_PROJECT_ID` (`personal-agent` /
  `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc`) or environment variables
  (`$ANTIGRAVITY_PROJECT_ID`, `$PROJECT_ID`, `$AGY_PROJECT_ID`).

## Sidecars & Skills
- **Scheduled Sidecars:** `model-updater`
  (`sidecars/model-updater/sidecar.json`) runs daily at 15:00 UTC to update
  Gemini model defaults; `submodule-updater` runs weekly (Mondays 15:00 UTC) to
  update vendored agent skills.
- **Slack Integration:** `sidecars/slack-chat/` bridges Slack Socket Mode to
  `agentapi` via `AgentApiBridge`, mapping `thread_ts` to `conversation_id` and
  backfilling via `conversations_replies`. Validated by
  `skills/prep-slack-chat-sidecar` for required tokens (`SLACK_BOT_TOKEN`,
  `SLACK_APP_TOKEN`) and dependencies.
- **Workflow & Chat Skills:** `self-review-commit` performs pre-commit code
  reviews with a `Model="pro"` reviewer subagent; `ollama-chat` queries Gemma 4
  models at `https://ollama.klinker-cabin.computer` via Bearer auth
  (`$OLLAMA_API_KEY`); `notify-the-user` sends Slack webhook notifications.
- **Memory Skills:** `lookup-memory` and `save-memory` provide on-demand memory
  retrieval and persistence in `$MEMORY_DIRECTORY`.
