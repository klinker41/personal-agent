---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-10-06T00:36:54.024737+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- **Daemon IPC & Hub State**: The Web UI interacts via local RPC and WebSockets
  with the compiled `agy` daemon. Connect-RPC endpoint `SetCloudCodeURL`
  strictly requires parameter `url` (never `cloudCodeUrl`); mismatches leave it
  empty, crashing the session until a container restart with
  `Post "/v1internal:loadCodeAssist": unsupported protocol scheme ""`. The
  hub stream `StreamAgentStateUpdates` emits plain JSON snapshots of steps with
  incrementally growing thinking text, avoiding binary protobuf diff decoding.
- **Cloud Code Protocol & Translation Proxy**: `agy` connects to internal Cloud
  Code endpoints (`/v1internal:streamGenerateContent` and
  `/v1internal:fetchAvailableModels`) via HTTP/2 Protobuf
  (`JetskiService`/`GetChatMessageRequest`). Integrating external LLMs requires
  a bidirectional proxy that formats streaming SSE chunks into
  `{"response": {"candidates": [...]}}` (not Gemini AI Studio schemas) and
  sanitizes arguments when models omit required fields (e.g., enforcing
  `Overwrite: true` and mandating `UserFacing` in `ArtifactMetadata` for
  `write_to_file`).
- **Model Routing & Stream Execution**: Placeholder IDs in M500–M649 (e.g.,
  M599, M605) denote built-in models; proxies must route unregistered IDs in
  this range to Cloud Code rather than custom sessions. `agy` mandates a single
  unified stream (`thought -> text -> functionCall`) with tools attached nearly
  every turn; split-model routing or fallback delegation (e.g., to Gemini Flash)
  bypasses model reasoning and introduces severe latency.
