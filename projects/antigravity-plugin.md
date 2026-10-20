---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-10-20T06:36:37.175208+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Repository at `/workspace/antigravity-plugin` (configured in
`~/.gemini/config/plugins.json`, tracking `main` from
`git@github.com:klinker41/antigravity-plugin.git`). Houses rules, skills,
lifecycle hooks (`hooks.json`), sidecars (`sidecar.json`), MCP configuration
(`mcp_config.json` in plugin root, not global directory), and
`vendor/agent-skills`.

## Development Rules & Invariants
- **Container & Web Services:** Protect container runtime by never killing
  `auth-proxy.js`, infra ports (4400, 4402, 4405, 7681, 8080), or executing
  blanket `pkill bun` (`rules/docker-container-environment.md`). Web apps must
  use Bun, Hono, minimal dependencies (`rules/web-app-architecture.md`), and
  port 4401 routed to `https://prototype.klinker-cabin.computer`
  (`rules/web-service-port.md`).
- **Coding, Review & Model Tiers:** Enforce kebab-case names, 80-column Markdown
  wrapping, minimal diffs (`rules/simplify-changes.md`), and README updates
  (`rules/readme-updates.md`). Subagents use `Model: "flash"` for coding/tests
  (`rules/coding-subagent-model.md`) and `Model: "pro"` for self-reviews.
- **Git Quality Gates:** All non-verbose tests must pass (`rules/tests.md`,
  `rules/non-verbose-tests.md`), commits must be secret-free
  (`rules/no-secrets-in-commits.md`) and pass `self-review-commit`
  (`rules/self-review-before-commit.md`). Never run `git push` without explicit
  user approval (`rules/git-push.md`).

## Memory System (`sidecars/memory-daemon`)
- **Architecture & Injection:** Progressive disclosure store under
  `$MEMORY_DIRECTORY` (`profile.md`, `index.md`, `projects/`, `knowledge/`, and
  tiered `daily/`, `monthly/`, `yearly/` chronicles) accessed via
  `lookup-memory` and `save-memory`. First-turn memory is injected when
  `initialNumSteps == 0` and `invocationNum == 1` (`hooks/inject_memory.py`).
- **Nightly Maintenance Pipeline:** Automated sequence in `memory-daemon`:
  00:00 Dreamer session extraction (watermarks in `state.json`, topic resolution
  via `get_existing_topics`, subagent/rule filtering, ephemeral session purge);
  00:30 Compactor chronicle consolidation and semantic refinement
  (`compactor.py`); 01:00 Git sync with secret scrubbing (`git_sync.py`).

## Sidecars & Platform Integrations
- **Agent API Bridge (`utils/memory_utils.py`):** Subprocess runner with auth
  retries resolving projects in `~/.gemini/config/projects/` by name, UUID,
  or path (defaults to `personal-agent`, UUID
  `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc` or `$ANTIGRAVITY_PROJECT_ID`). Cleans
  env (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
  `ANTIGRAVITY_TRAJECTORY_ID`) and extracts `ANTIGRAVITY_LS_ADDRESS` and CSRF
  tokens (`window.__APP_CONFIG__.csrfToken`) from `cli.log` and runtime state.
- **Scheduled Tasks & Slack Daemon (`sidecar.json`):** Recurring cron jobs run
  daily Gemini `model-updater` (15:00 UTC) and weekly `vendor/agent-skills`
  `submodule-updater` (Mondays 15:00 UTC). Slack daemon (`sidecars/slack-chat/`,
  validated via `prep-slack-chat-sidecar`) bridges Socket Mode threads to
  `agentapi` (`agy agentapi new-conversation --model=<tier>`) using model tiers
  configured in `sidecar.json` (e.g., `model: 'pro'`) with backfill; outbound
  alerts use `notify-the-user`.
- **External AI Skills:** `ollama-chat` queries Gemma models hosted at
  `https://ollama.klinker-cabin.computer` using `$OLLAMA_API_KEY`.
