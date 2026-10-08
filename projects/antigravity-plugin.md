---
topic: antigravity-plugin
category: project
tags: [antigravity, plugin, sidecars, skills, rules]
updated_at: 2026-10-08T00:34:37.814435+00:00
confidence: 1.0
---

# Project Context: Antigravity Plugin

Repository at `/workspace/antigravity-plugin` (`~/.gemini/config/plugins.json`,
tracking `main` from `git@github.com:klinker41/antigravity-plugin.git`).
Houses rules, skills, lifecycle hooks (`hooks.json`), sidecars
(`sidecar.json`), and `vendor/agent-skills`.

## Development Rules & Invariants
- **Container Safety:** Prohibits terminating the proxy (`auth-proxy.js`),
  blanket Bun kills (`pkill bun`), and killing processes on infra ports
  (4400, 4402, 4405, 7681, 8080; `rules/docker-container-environment.md`).
- **Web Applications:** Bun runtime/package manager, Hono framework, minimal
  dependencies (`rules/web-app-architecture.md`), and port 4401 routed via
  `https://prototype.klinker-cabin.computer` (`rules/web-service-port.md`).
- **Coding Standards:** Kebab-case naming for rules/skills, 80-char Markdown
  wrap, minimal diffs (`rules/simplify-changes.md`), README updates on
  structural changes, and `Model: "flash"` for coding/test subagents
  (`rules/coding-subagent-model.md`).
- **Git & Quality Gates:** Passing non-verbose tests
  (`rules/non-verbose-tests.md`, `rules/tests.md`), zero secret leaks
  (`rules/no-secrets-in-commits.md`), mandatory `self-review-commit` loop
  (`Model: "pro"`), and explicit user approval before `git push`
  (`rules/git-push.md`).

## Memory System (`sidecars/memory-daemon`)
- **Structure & Access:** Progressive disclosure store at `$MEMORY_DIRECTORY`
  (`profile.md`, `index.md`, `projects/`, `knowledge/`, and tiered
  chronicles in `daily/`, `monthly/`, `yearly/`), accessed via
  `lookup-memory` and `save-memory`.
- **Lifecycle & Scheduling:** Turn-1 injection on `initialNumSteps == 0` and
  `invocationNum == 1` (`hooks/inject_memory.py`). Nightly maintenance:
  - `00:00`: Dreamer extraction (persists watermarks to `state.json`, checks
    topics via `memory_utils.get_existing_topics`, skips subagents and
    `rules/*.md`, purges ephemeral sessions).
  - `00:30`: Tiered chronicle compaction and overgrown note refinement.
  - `01:00`: Git sync with secret scrubbing.

## Sidecars & Integrations
- **Agent API Bridge (`utils/memory_utils.py`):** `AgentApiBridge` runs
  `agentapi` subprocesses with auth retry, resolving projects in
  `~/.gemini/config/projects/` by name, UUID, or path (defaults to
  `personal-agent`, UUID `6b1d3dc5-a020-4710-94f5-79b34fc1b9fc` or
  `$ANTIGRAVITY_PROJECT_ID`). Strips caller session environment variables
  (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
  `ANTIGRAVITY_TRAJECTORY_ID`); recovers `ANTIGRAVITY_LS_ADDRESS` and CSRF
  tokens (`window.__APP_CONFIG__.csrfToken`) from `cli.log`/runtime state.
- **Scheduled Maintenance (`sidecar.json`):** Recurring `agentapi` jobs
  (`new-conversation --model`): daily `model-updater` (Gemini defaults at
  15:00 UTC) and weekly `submodule-updater` (`vendor/agent-skills` Mondays at
  15:00 UTC).
- **Slack Integration (`sidecars/slack-chat/`):** Connects Slack Socket Mode
  to `agentapi` via `AgentApiBridge`, mapping `thread_ts` to conversation
  IDs with `conversations_replies` backfill. Verified by
  `prep-slack-chat-sidecar` (`SLACK_BOT_TOKEN`, `SLACK_APP_TOKEN`); alerts
  sent via `notify-the-user`.
- **External AI Skills:** `ollama-chat` queries Gemma models hosted at
  `https://ollama.klinker-cabin.computer` using `$OLLAMA_API_KEY`.
