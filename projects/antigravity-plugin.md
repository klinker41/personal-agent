---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-08-30T11:30:08.722025+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Custom plugin bundle for Google Antigravity packaging rules, skills, sidecars,
and lifecycle hooks. Registered locally in `~/.gemini/config/plugins.json`
pointing to `/workspace/antigravity-plugin`. Remote repository configured at
`git@github.com:klinker41/antigravity-plugin.git` tracking `main`.

## Key Architectural Invariants & Rules
- **Web Service Port:** All web services must run on port 4401, accessible via
  `https://prototype.klinker-cabin.computer` (`rules/web-service-port.md`).
- **Sidecar Lifecycle:** Background services run as daemons managed via
  `sidecar.json`.
- **Git Push Policy:** Always use `git commit` after self-review; never run
  `git push` automatically without explicit confirmation (`rules/git-push.md`).
- **Code Simplification:** `rules/simplify-changes.md` is `always_on`, mandating
  minimal diffs, zero extraneous changes, code deduplication, and clarity.
- **Line Wrapping:** Strict 80-character maximum per line in `.md` files.

## Memory Architecture & Daemon (`sidecars/memory-daemon`)
- **Storage & Structure:** Backed by a Git repo at `$MEMORY_DIRECTORY` using
  progressive disclosure (`profile.md`, `index.md`, `projects/`, `knowledge/`,
  and tiered daily/monthly/yearly chronicles).
- **Hybrid Processing:** Deterministic Python handles security scrubbing,
  hooks (`hooks/inject_memory.py` for pre-invocation injection), and Git sync;
  `agentapi` handles LLM fact synthesis and compaction.
- **Schedule:** Automated nightly runs at 02:00 (dreaming/extraction), 02:30
  (tiered compaction), and 03:00 (git commit and push).
- **Extraction Logic:** Tracks incremental processing per conversation using
  step-index watermarking in `state.json` rather than calendar timestamps.
  Filters sub-agents via routing headers (e.g., 'Message from Root Agent') and
  role definitions. Scans `rules/*.md` duri
<truncated 291 bytes>
clutter.

## Shared Utilities (`utils/memory_utils.py`)
- **Dynamic LS Discovery:** Resolves live `ANTIGRAVITY_LS_ADDRESS` and
  `ANTIGRAVITY_CSRF_TOKEN` by probing runtime files and active `cli.log` when
  environment variables become stale.
- **AgentApiBridge:** Centralized bridge executing `agentapi` subprocesses with
  dynamically resolved live LS credentials.
- **Project ID Resolution:** Resolves project identifiers against
  `~/.gemini/config/projects/` by name, UUID, or path, falling back to
  `$ANTIGRAVITY_PROJECT_ID`, `$PROJECT_ID`, and `$AGY_PROJECT_ID`.

## Sidecars & Skills
- **Slack Chat Sidecar (`sidecars/slack-chat/`):**
  - Bridges Slack Socket Mode to Antigravity Agent API (`agy agentapi`) using
    `AgentApiBridge` and maps `thread_ts` to `conversation_id` in JSON state.
  - Associated with `heartbeat` project ID
    `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc`.
  - Prepends context from `conversations_replies` on unmapped threads
    (requires `channels:history`, `groups:history`, and `mpim:history` scopes).
  - Automatically checks and installs `requirements.txt` on startup via
    `ensure_requirements()` using `--break-system-packages` and
    `importlib.invalidate_caches()`.
- **Prep Slack Chat Sidecar (`skills/prep-slack-chat-sidecar`):** Validates
  Slack tokens (`SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`), dependencies, and
  project registrations via `check_readiness.py`.
- **Self-Review Commit (`skills/self-review-commit`):** Spawns a reviewer
  subagent with `Model="pro"` for pre-commit evaluations. Requires outputting
  a summary of committed changes and resolved findings after LGTM.
- **Ollama Chat (`skills/ollama-chat`):** Pure Python 3 CLI client
  (`scripts/ollama_client.py`) using `urllib.request` and `$OLLAMA_API_KEY` for
  Bearer auth to query remote Ollama at `https://ollama.klinker-cabin.computer`
  hosting Gemma 4 models (`gemma4:26b`, `gemma4:12b`, `gemma4:e4b`, and
  `gemma4:e2b`).
- **Memory Skills:** `lookup-memory` and `save-memory` provide on-demand memory
  retrieval and updates.
