---
topic: antigravity-agentapi-bridge
category: knowledge
tags: [knowledge, antigravity, agentapi, language-server, transcript-polling]
updated_at: 2026-10-30T18:59:00+00:00
compacted_at: 2026-10-30T18:59:00+00:00
confidence: 0.95
---

# Knowledge: Antigravity AgentAPI & Language Server Bridge

- When `ANTIGRAVITY_AGENTAPI_EXE` points to the `agy` binary
  (`/home/developer/.local/bin/agy`), invocations must include the `agentapi`
  subcommand (e.g., `agy agentapi new-conversation`) rather than invoking
  subcommands directly on `agy`.
- Invoking `agentapi` from background daemon processes requires injecting
  `ANTIGRAVITY_LS_ADDRESS` (`127.0.0.1:<PORT>`) and `ANTIGRAVITY_CSRF_TOKEN`
  into the execution environment, along with a valid workspace Project UUID in
  `PROJECT_ID` / `ANTIGRAVITY_PROJECT_ID` (mapped in
  `~/.gemini/config/projects/<UUID>.json`). Passing an empty or raw project name
  string, or a mismatched project UUID, triggers `project_id is required` or
  `PermissionDenied` errors.
- Daemon background tasks and subprocess bridges communicating with the
  Antigravity Language Server must clear ambient conversation metadata
  (`ANTIGRAVITY_SOURCE_METADATA`, `ANTIGRAVITY_CONVERSATION_ID`,
  `ANTIGRAVITY_TRAJECTORY_ID`) to avoid cross-project context errors and source
  project mismatch conflicts when targeting other workspaces.
- The `agy` CLI supports `--hub-port <port>` in `--remote-control` mode to bind
  the hub/language server to a deterministic port (e.g., `4402`). Because CSRF
  tokens (and dynamic LS ports) change across restarts, `AgentApiBridge` checks
  socket liveness, reads active values from `ls_address` / `cli.log` /
  `~/.gemini/antigravity/sidecar_data/`, and handles
  `Unauthenticated (invalid CSRF token)` errors by extracting
  `window.__APP_CONFIG__.csrfToken` from `http://127.0.0.1:4402/` and retrying.
- `agy agentapi new-conversation` accepts `--model=<flash_lite|flash|pro>` using
  abstract tier keywords only; passing explicit model IDs like `argon` fails
  with `no available models found for tier <name>`. Upstream Google
  `TieredModelConfig` maps `pro` to `gemini-3.8-flash-tiered`
  (`MODEL_PLACEHOLDER_M322`), so the translation proxy rewrites it to Gemini 4
  Argon (`MODEL_PLACEHOLDER_M417` / `MODEL_PLACEHOLDER_M418`).
- `agentapi new-conversation` creates persistent directories under
  `~/.gemini/antigravity-cli/brain/<uuid>` that appear as standard conversations
  unless explicitly archived or deleted. Sub-agent transcripts can be detected
  by checking message routing headers like `Message from Root Agent`.
- Programmatic bridges monitor `agentapi` execution by watching the
  conversation's `transcript.jsonl` for completed `PLANNER_RESPONSE` entries,
  maintaining file byte offsets to parse delta lines incrementally instead of
  performing `O(N^2)` full-file re-reads.
