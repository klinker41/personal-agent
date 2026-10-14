---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-10-14T00:34:57.310582+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Repository at `/workspace/antigravity-plugin`
(`~/.gemini/config/plugins.json`, tracking `main` from
`git@github.com:klinker41/antigravity-plugin.git`). Houses rules, skills,
lifecycle hooks (`hooks.json`), sidecars (`sidecar.json`), and
`vendor/agent-skills`.

## Development Rules & Invariants
- **Container Safety & Web Apps:** Never kill `auth-proxy.js`, infrastructure
  ports (4400, 4402, 4405, 7681, 8080), or run blanket `pkill bun`
  (`rules/docker-container-environment.md`). Web apps require Bun, Hono,
  minimal dependencies (`rules/web-app-architecture.md`), and port 4401 routed
  via `https://prototype.klinker-cabin.computer` (`rules/web-service-port.md`).
- **Standards & Model Selection:** Rules and skills use kebab-case. Wrap
  Markdown at 80 columns, keep diffs minimal (`rules/simplify-changes.md`), and
  update README on structural changes. Subagents use `Model: "flash"` for
  coding/testing (`rules/coding-subagent-model.md`) and `Model: "pro"` for
  self-reviews.
- **Git & Quality Gates:** All non-verbose tests must pass before commit
  (`rules/tests.md`, `rules/non-verbose-tests.md`), with zero secrets
  (`rules/no-secrets-in-commits.md`). Requires `self-review-commit` before
  creating commits and explicit user approval before `git push`
  (`rules/git-push.md`).

## Memory System (`sidecars/memory-daemon`)
- **Structure & Injection:** Progressive disclosure store at `$MEMORY_DIRECTORY`
  (`profile.md`, `index.md`, `projects/`, `knowledge/`, tiered `daily/`,
  `monthly/`, `yearly/` chronicles) queried via `lookup-memory` and
  `save-memory`. Injects turn-1 memory when `initialNumSteps == 0` and
  `invocationNum == 1` (`hooks/inject_memory.py`).
- **Nightly Maintenance Pipeline:**
  - `00:00`: Dreamer extraction (saves watermarks to `state.json`, checks
    topics via `memory_utils.get_existing_topics`, skips subagents and
    `rules/*.md`, purges ephemeral sessions).
  - `00:30`: Compactor tiered chronicle consolidation and overgrown note
    refinement (`compactor.py`).
  - `01:00`: Git sync with secret scrubbing (`git_sync.py`).

## Sidecars & Integrations
- **Agent API Bridge (`utils/memory_utils.py`):** Runs `agentapi` subprocesses
  with auth retries, resolving projects in `~/.gemini/config/projects/` by
  name, UUID, or path (defaults to `personal-agent`, UUID
  `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc` or `$ANTIGRAVITY_PROJECT_ID`).
  Sanitizes environment (`ANTIGRAVITY_SOURCE_METADATA`,
  `ANTIGRAVITY_CONVERSATION_ID`, `ANTIGRAVITY_TRAJECTORY_ID`); extracts
  `ANTIGRAVITY_LS_ADDRESS` and CSRF tokens (`window.__APP_CONFIG__.csrfToken`)
  from `cli.log`/runtime state.
- **Sidecars & Scheduled Tasks (`sidecar.json`):** Recurring `agentapi` jobs
  run daily Gemini `model-updater` (15:00 UTC) and weekly `submodule-updater`
  for `vendor/agent-skills` (Mondays at 15:00 UTC). Slack daemon
  (`sidecars/slack-chat/`) maps Socket Mode threads to `agentapi` conversations
  with backfill (validated via `prep-slack-chat-sidecar`; notifications via
  `notify-the-user`).
- **External AI Skills:** `ollama-chat` queries Gemma models hosted at
  `https://ollama.klinker-cabin.computer` using `$OLLAMA_API_KEY`.
