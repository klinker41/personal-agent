---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity, architecture, hooks, plugins, subagents]
updated_at: 2026-10-30T18:59:00+00:00
compacted_at: 2026-10-30T18:59:00+00:00
confidence: 0.95
---

# Knowledge: Antigravity Architecture, Storage & Customizations

- **Daemon IPC & Hub State**: The Web UI communicates with `agy` via Connect-RPC
  and WebSockets. `StreamAgentStateUpdates` emits plain JSON step snapshots with
  incremental thinking text, avoiding binary protobuf diff decoding. RPC
  `SetCloudCodeURL` strictly requires parameter `url` (not `cloudCodeUrl`);
  mismatches leave it empty, crashing the session with `Post
  "/v1internal:loadCodeAssist": unsupported protocol scheme ""` and requiring a
  container restart.
- **Cloud Code Protocol & Translation Proxy**: `agy` connects to internal Cloud
  Code endpoints (`/v1internal:streamGenerateContent` and
  `/v1internal:fetchAvailableModels`) via HTTP/2 Protobuf (`JetskiService` /
  `GetChatMessageRequest`). External LLM integration requires a bidirectional
  proxy translating streaming SSE chunks into
  `{"response": {"candidates": [...]}}` (not Gemini AI Studio schemas) and
  injecting missing required tool parameters (e.g., `Overwrite: true` and
  `ArtifactMetadata.UserFacing` for `write_to_file`).
- **Model Routing & Stream Execution**: IDs in range M500–M649 (e.g., M599,
  M605) denote built-in models; proxies must route unregistered IDs in this
  range directly to Cloud Code. `agy` enforces a single unified stream
  (`thought -> text -> functionCall`) with tools attached nearly every turn;
  split-model routing or fallback delegation bypasses reasoning and adds severe
  latency.
- **OAuth & API Scopes**: Standalone Google OAuth tokens reside at
  `~/.gemini/jetski-standalone-oauth-token`. Scopes are strictly restricted to
  GCP, AI Code, CCLog, experiments/configs, and identity (`cloud-platform`,
  `aicode`, `cclog`, `experimentsandconfigs`, `openid`, `email`, `profile`).
  Workspace scopes (`gmail.readonly`, `calendar.readonly`) are excluded; calling
  Gmail or Calendar APIs returns `403 Insufficient Permission`.
- **State & Trajectory Persistence**: Antigravity CLI keeps its state file
  `antigravity_state.pbtxt` and runtime logs `cli.log` in
  `~/.gemini/antigravity-cli/`. Overwriting `antigravity_state.pbtxt` resets
  `installation_uuid` and schema migration markers, breaking trajectory loading.
  Conversations are persisted across three locations: transcripts/artifacts in
  `~/.gemini/antigravity-cli/brain/<id>/`, SQLite records (`.db`, `-wal`,
  `-shm`) in `~/.gemini/antigravity-cli/conversations/<id>.db`, and sidebar
  titles in `~/.gemini/antigravity-cli/annotations/<id>.pbtxt` (with summaries
  in `conversation_summaries.db` and `cache/conversation_metadata.json`). Full
  deletion requires purging all three.
- **Customizations, Hooks & Subagents**:
  - Local plugins are registered in `~/.gemini/config/plugins.json` under an
    `entries` array of `{ "path": ... }` objects, with MCP server configs at
    `mcp_config.json` in the plugin root.
  - In Antigravity `PreInvocation` hooks, `USER_INPUT` is already recorded at
    `step_index: 0` when the hook fires (`initialNumSteps` is `1` or `2` on Turn
    1); checking `invocationNum == 1` and `initialNumSteps <= 2` alongside
    verifying at most 1 user input step in `transcript.jsonl` ensures context is
    injected only on the initial conversation turn.
  - `invoke_subagent` supports `Model` tiers (`pro`, `flash`, `flash_lite`,
    `inherit`). Subagent rules should condition model selection on subagent
    invocation rather than mandating delegation.
  - Antigravity sandbox and security settings include `enableTerminalSandbox`,
    `autoExecutionPolicy`
    (`CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`),
    `nonWorkspaceFileAccessPolicy` (`AGENT_SETTING_POLICY_DENY`), and
    `artifactReviewMode` (`ARTIFACT_REVIEW_MODE_ALWAYS`).
  - The artifact renderer supports `flowchart`/`graph`, `stateDiagram-v2`,
    `sequenceDiagram`, `classDiagram`, `erDiagram`, and `xychart-beta` Mermaid
    diagrams, but does not support `gantt` charts.
- **Telemetry Blocking**: Gemini inference (`cloudaicompanion.googleapis.com`,
  `cloudcode-pa.googleapis.com`, `generativelanguage.googleapis.com`) and auth
  (`accounts.google.com`, `oauth2.googleapis.com`) are separate from telemetry
  domains (`firebaselogging-pa.googleapis.com`, `feedback-pa.googleapis.com`,
  `cloudtrace.googleapis.com`, `clouderrorreporting.googleapis.com`), which can
  be mapped to `0.0.0.0` in `/etc/hosts` alongside `DO_NOT_TRACK=1` and
  `OTEL_SDK_DISABLED=true`.
- **Web UI & Mobile Multiplexing**:
  - The web UI stores the active project selection in browser `localStorage` per
    client, requiring project selection on new devices before scoped
    conversations appear.
  - Appending `useWebSocket=true` to the Antigravity URL enables WebSocket RPC
    multiplexing over `/connect-websocket`, avoiding browser HTTP/1.1
    6-connection per-origin stalls from 5+ concurrent streaming RPCs.
  - When spoofing a desktop `navigator.userAgent` on foldable mobile devices to
    force split-pane layout, overriding `shiftKey = true` during capture-phase
    `keydown` restores mobile newline behavior in Lexical.
  - The mobile gateway multiplexes multiple conversation streams and list
    updates over a single device WebSocket connection using per-topic sequence
    numbers and a replay buffer.
