---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-10-20T06:40:22.525181+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- **Daemon IPC & Hub State**: The Web UI communicates with `agy` via
  Connect-RPC and WebSockets. `StreamAgentStateUpdates` emits plain JSON step
  snapshots with incremental thinking text, avoiding binary protobuf diff
  decoding. RPC `SetCloudCodeURL` strictly requires parameter `url` (not
  `cloudCodeUrl`); mismatches leave it empty, crashing the session with
  `Post "/v1internal:loadCodeAssist": unsupported protocol scheme ""` and
  requiring a container restart.
- **Cloud Code Protocol & Translation Proxy**: `agy` connects to internal
  Cloud Code endpoints (`/v1internal:streamGenerateContent` and
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
  split-model routing or fallback delegation (e.g., to Gemini Flash) bypasses
  reasoning and introduces severe latency.
- **OAuth & API Scopes**: Standalone Google OAuth tokens reside at
  `~/.gemini/jetski-standalone-oauth-token`. Scopes are strictly restricted to
  GCP, AI Code, CCLog, experiments/configs, and identity (`cloud-platform`,
  `aicode`, `cclog`, `experimentsandconfigs`, `openid`, `email`, `profile`).
  Workspace scopes (`gmail.readonly`, `calendar.readonly`) are excluded;
  calling Gmail or Calendar APIs returns 403 Insufficient Permission.
