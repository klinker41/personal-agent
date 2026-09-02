---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-09-02T00:31:17.315517+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Bundle packaging rules, skills, sidecars, and lifecycle hooks for Google
Antigravity. Located at `/workspace/antigravity-plugin`, registered locally
in `~/.gemini/config/plugins.json`, and tracks remote `main` branch at
`git@github.com:klinker41/antigravity-plugin.git`. Tracks agent skills via the
`vendor/agent-skills` git submodule.

## Key Invariants & Rules
- **Web Service Port:** All web services must run on port 4401, accessible via
  `https://prototype.klinker-cabin.computer` (`rules/web-service-port.md`).
- **Sidecars:** Run as managed background daemons declared in `sidecar.json`.
- **Git Push Policy:** Always run `git commit` after self-review; never run
  `git push` automatically without explicit user confirmation
  (`rules/git-push.md`).
- **Code Simplification:** `rules/simplify-changes.md` is `always_on`,
  mandating minimal diffs, zero extraneous edits, deduplication, and clarity.
- **Formatting:** Strict 80-character maximum line wrapping in `.md` files.

## Memory Architecture & Daemon (`sidecars/memory-daemon`)
- **Structure & Storage:** Progressive disclosure repository at
  `$MEMORY_DIRECTORY` (`profile.md`, `index.md`, `projects/`, `knowledge/`, and
  tiered `daily/`, `monthly/`, `yearly/` chronicles).
- **Hybrid Processing:** Deterministic Python handles secret scrubbing, Git
  sync, and lifecycle hooks (`hooks/inject_memory.py`); `agentapi` handles LLM
  extraction, synthesis, and compaction.
- **Hook Injection:** `hooks/inject_memory.py` injects `profile.md` on turn 1
  (`initialNumSteps == 0` and `invocationNum == 1`) and checks transcript
  history to avoid redundant context on follow-ups.
- **Synthesis & Cataloging:** `dreamer.py` uses
  `utils.memory_utils.get_existing_topics` to inject catalogs of existing
  knowledge topics and project names into LLM prompts, preventing duplicate
  files. 
<truncated 253 bytes>
  00:30, and Git sync at 01:00 local time.
- **Extraction & Pruning:** Watermarks step indices in `state.json` for
  incremental processing. Skips subagents and omits rules already covered in
  `rules/*.md`. Purges internal ephemeral conversation folders after runs.

## Shared Utilities (`utils/memory_utils.py`)
- **Dynamic LS Discovery:** Probes runtime state files and active `cli.log` for
  live `ANTIGRAVITY_LS_ADDRESS` and `ANTIGRAVITY_CSRF_TOKEN` when env vars
  become stale.
- **AgentApiBridge:** Subprocess runner for `agentapi`. Strips caller metadata
  (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
  `ANTIGRAVITY_TRAJECTORY_ID`) to avoid project mismatch errors. Dynamically
  fetches CSRF tokens from the web hub endpoint
  (`window.__APP_CONFIG__.csrfToken`) and retries on auth failures.
- **Project ID Resolution:** Resolves IDs against `~/.gemini/config/projects/`
  by name, UUID, or path. Falls back to `DEFAULT_PROJECT_ID` (`personal-agent`
  / `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc`) or environment variables
  (`$ANTIGRAVITY_PROJECT_ID`, `$PROJECT_ID`, `$AGY_PROJECT_ID`).

## Sidecars & Skills
- **Slack Chat Sidecar (`sidecars/slack-chat/`):** Bridges Slack Socket Mode to
  `agentapi` via `AgentApiBridge`, maps `thread_ts` to `conversation_id`,
  backfills context via `conversations_replies` on unmapped threads, and
  auto-installs `requirements.txt` on startup.
- **Prep Slack Chat (`skills/prep-slack-chat-sidecar`):** Validates Slack
  tokens (`SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`), dependencies, and project
  setup via `check_readiness.py`.
- **Self-Review Commit (`skills/self-review-commit`):** Runs pre-commit review
  with a `Model="pro"` reviewer subagent and summarizes resolved findings.
- **Ollama Chat (`skills/ollama-chat`):** Pure Python CLI client
  (`scripts/ollama_client.py`) using Bearer auth (`$OLLAMA_API_KEY`) to query
  Gemma 4 models at `https://ollama.klinker-cabin.computer`.
- **Memory Skills:** `lookup-memory` and `save-memory` provide on-demand
  retrieval and updates.
