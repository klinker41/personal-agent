---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-10-13T00:35:44.796751+00:00
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
  (`rules/docker-container-environment.md`). Web apps must use Bun runtime,
  Hono framework, minimal dependencies (`rules/web-app-architecture.md`), and
  port 4401 routed via `https://prototype.klinker-cabin.computer`
  (`rules/web-service-port.md`).
- **Coding Standards & Models:** Use kebab-case for rules/skills, 80-character
  max Markdown wrap, minimal diffs (`rules/simplify-changes.md`), and README
  updates on structural changes. Configure `Model: "flash"` for coding/testing
  subagents (`rules/coding-subagent-model.md`) and `Model: "pro"` for
  self-reviews.
- **Git & Quality Gates:** All non-verbose tests must pass before commit
  (`rules/tests.md`, `rules/non-verbose-tests.md`), with zero secrets permitted
  (`rules/no-secrets-in-commits.md`). Requires mandatory `self-review-commit`
  loop and explicit user approval before `git push` (`rules/git-push.md`).

## Memory System (`sidecars/memory-daemon`)
- **Structure & Access:** Progressive disclosure store at `$MEMORY_DIRECTORY`
  (`profile.md`, `index.md`, `projects/`, `knowledge/`, tiered `daily/`,
  `monthly/`, `yearly/` chronicles) accessed via `lookup-memory` and
  `save-memory`. Injects turn-1 memory when `initialNumSteps == 0` and
  `invocationNum == 1` (`hooks/inject_memory.py`).
- **Nightly Maintenance:**
  - `00:00`: Dreamer extraction (persists watermarks to `state.json`, checks
    topics via `memory_utils.get_existing_topics`, skips subagents and
    `rules/*.md`, purges ephemeral sessions).
  - `00:30`: Tiered chronicle compaction and overgrown semantic note
    refinement (`compactor.py`).
  - `01:00`: Git sync with secret scrubbing (`git_sync.py`).

## Sidecars & Integrations
- **Agent API Bridge (`utils/memory_utils.py`):** Runs `agentapi` subprocesses
  with auth retry, resolving projects in `~/.gemini/config/projects/` by name,
  UUID, or path (defaults to `personal-agent`, UUID
  `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc` or `$ANTIGRAVITY_PROJECT_ID`).
  Sanitizes session environment (`ANTIGRAVITY_SOURCE_METADATA`,
  `ANTIGRAVITY_CONVERSATION_ID`, `ANTIGRAVITY_TRAJECTORY_ID`); extracts
  `ANTIGRAVITY_LS_ADDRESS` and CSRF tokens (`window.__APP_CONFIG__.csrfToken`)
  from `cli.log`/runtime state.
- **Sidecars & Scheduled Tasks (`sidecar.json`):** Recurring `agentapi` jobs
  (`new-conversation --model`) execute daily Gemini `model-updater` (15:00 UTC)
  and weekly `vendor/agent-skills` `submodule-updater` (Mondays at 15:00 UTC).
  Slack daemon (`sidecars/slack-chat/`) maps Socket Mode threads to `agentapi`
  conversations with backfill (verified via `prep-slack-chat-sidecar`; alerts
  sent via `notify-the-user`).
- **External AI Skills:** `ollama-chat` queries Gemma models hosted at
  `https://ollama.klinker-cabin.computer` using `$OLLAMA_API_KEY`.
