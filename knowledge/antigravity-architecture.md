---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity, architecture, hooks, plugins, subagents]
updated_at: 2026-11-21T07:39:07.228725+00:00
compacted_at: 2026-11-21T07:39:07.228725+00:00
confidence: 0.95
---

# Knowledge: Antigravity Architecture, Storage & Customizations

- **Daemon IPC & Hub State**: The Web UI communicates with `agy` via
  Connect-RPC and WebSockets. `StreamAgentStateUpdates` emits plain JSON
  step snapshots with incremental thinking text, avoiding binary protobuf
  diff decoding. RPC `SetCloudCodeURL` strictly requires parameter `url`
  (not `cloudCodeUrl`); mismatches leave it empty, crashing the session
  with `Post "/v1internal:loadCodeAssist": unsupported protocol scheme ""`
  and requiring a container restart.
- **Cloud Code Protocol & Translation Proxy**: `agy` connects to internal
  Cloud Code endpoints (`/v1internal:streamGenerateContent` and
  `/v1internal:fetchAvailableModels`) via HTTP/2 Protobuf (`JetskiService`
  / `GetChatMessageRequest`). External LLM integration requires a
  bidirectional proxy translating streaming SSE chunks into
  `{"response": {"candidates": [...]}}` (not Gemini AI Studio schemas) and
  injecting missing required tool parameters (e.g., `Overwrite: true` and
  `ArtifactMetadata.UserFacing` for `write_to_file`).
- **Model Routing, Tiers & Stream Execution**: Built-in model IDs span
  M500–M649 (e.g., M599, M605); proxies must route unregistered IDs in this
  range directly to Cloud Code. `fetchAvailableModels` /
  `GetAvailableModels` defines `tieredModelIds` (`pro`, `flash`,
  `flashLite`, with upstream defaults like `gemini-3.8-flash-tiered` and
  `gemini-3.5-flash-lite`); when `translation-proxy.js` rewrites
  `tieredModelIds`, in-container `GetAvailableModels` queries return active
  proxy overrides rather than raw upstream defaults unless no candidate ID
  matched the catalog. `agy` enforces a single unified stream
  (`thought -> text -> functionCall`) with tools attached nearly every
  turn; split-model routing or fallback delegation bypasses reasoning and
  adds severe latency.
- **OAuth & API Scopes**: Standalone Google OAuth tokens reside at
  `~/.gemini/jetski-standalone-oauth-token` and are restricted to scopes
  `cloud-platform`, `aicode`, `cclog`, `experimentsandconfigs`, `openid`,
  `email`, and `profile`. Workspace scopes (`gmail.readonly`,
  `calendar.readonly`) are excluded; Gmail or Calendar API calls return
  `403 Insufficient Permission`.
- **State & Trajectory Persistence**: Antigravity CLI stores state in
  `~/.gemini/antigravity-cli/antigravity_state.pbtxt` and runtime logs in
  `cli.log`; overwriting `antigravity_state.pbtxt` resets
  `installation_uuid` and schema migration markers, breaking trajectory
  loading. Conversations persist across three paths in
  `~/.gemini/antigravity-cli/`: transcripts/artifacts in `brain/<id>/`,
  SQLite files (`.db`, `-wal`, `-shm`) in `conversations/<id>.db`, and
  sidebar titles in `annotations/<id>.pbtxt` (plus summaries in
  `conversation_summaries.db` and `cache/conversation_metadata.json`). Full
  deletion requires purging all three.
- **Customizations, Hooks & Subagents**:
  - Local plugins register in `~/.gemini/config/plugins.json` under an
    `entries` array of `{ "path": ... }` objects, with MCP server configs
    at `mcp_config.json` in the plugin root.
  - In `PreInvocation` hooks, `USER_INPUT` is already recorded at
    `step_index: 0` when the hook fires (`initialNumSteps` is `1` or `2` on
    Turn 1); checking `invocationNum == 1`, `initialNumSteps <= 2`, and at
    most 1 user input step in `transcript.jsonl` restricts context
    injection to Turn 1.
  - `invoke_subagent` supports `Model` tiers (`pro`, `flash`, `flash_lite`,
    `inherit`); subagent rules should condition model selection on subagent
    invocation rather than mandating delegation.
  - Sandbox and security settings include `enableTerminalSandbox`,
    `autoExecutionPolicy`
    (`CASCADE_COMMANDS_AUTO_EXECUTION_PROCEED_IN_SANDBOX`),
    `nonWorkspaceFileAccessPolicy` (`AGENT_SETTING_POLICY_DENY`), and
    `artifactReviewMode` (`ARTIFACT_REVIEW_MODE_ALWAYS`).
  - Artifact Mermaid rendering supports `flowchart`/`graph`,
    `stateDiagram-v2`, `sequenceDiagram`, `classDiagram`, `erDiagram`, and
    `xychart-beta`, but not `gantt` charts.
- **Telemetry Blocking**: Inference (`cloudaicompanion.googleapis.com`,
  `cloudcode-pa.googleapis.com`, `generativelanguage.googleapis.com`) and
  auth (`accounts.google.com`, `oauth2.googleapis.com`) are isolated from
  telemetry domains (`firebaselogging-pa.googleapis.com`,
  `feedback-pa.googleapis.com`, `cloudtrace.googleapis.com`,
  `clouderrorreporting.googleapis.com`), which can be sinkholed to
  `0.0.0.0` in `/etc/hosts` alongside `DO_NOT_TRACK=1` and
  `OTEL_SDK_DISABLED=true`.
- **Web UI & Mobile Multiplexing**:
  - Active project selection is stored in browser `localStorage` per
    client, requiring selection on new devices before scoped conversations
    appear.
  - Appending `useWebSocket=true` to the URL multiplexes RPCs over
    `/connect-websocket`, avoiding browser HTTP/1.1 6-connection per-origin
    stalls from 5+ concurrent streaming RPCs.
  - When spoofing a desktop `navigator.userAgent` on foldable devices to
    force split-pane layout, overriding `shiftKey = true` during
    capture-phase `keydown` restores mobile newline behavior in Lexical.
  - The mobile gateway multiplexes conversation streams and list updates
    over a single device WebSocket using per-topic sequence numbers and a
    replay buffer.
