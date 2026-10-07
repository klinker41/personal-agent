---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-10-07T00:37:47.679252+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- **Daemon IPC & Hub State**: The Web UI interacts via Connect-RPC and
  WebSockets with the `agy` daemon. Endpoint `SetCloudCodeURL` strictly
  requires parameter `url` (never `cloudCodeUrl`); mismatches leave it empty
  and crash the session on `Post "/v1internal:loadCodeAssist": unsupported
  protocol scheme ""` until a container restart. Hub stream
  `StreamAgentStateUpdates` emits plain JSON step snapshots with
  incrementally growing thinking text, avoiding binary protobuf diff decoding.
- **Cloud Code Protocol & Translation Proxy**: `agy` connects to internal
  Cloud Code endpoints (`/v1internal:streamGenerateContent` and
  `/v1internal:fetchAvailableModels`) via HTTP/2 Protobuf
  (`JetskiService`/`GetChatMessageRequest`). Integrating external LLMs
  requires a bidirectional proxy that formats streaming SSE chunks into
  `{"response": {"candidates": [...]}}` (not Gemini AI Studio schemas) and
  sanitizes missing required arguments (e.g., enforcing `Overwrite: true` and
  mandating `UserFacing` in `ArtifactMetadata` for `write_to_file`).
- **Model Routing & Stream Execution**: Placeholder IDs in M500–M649 (e.g.,
  M599, M605) denote built-in models; proxies must route unregistered IDs
  in this range to Cloud Code rather than custom sessions. `agy` enforces a
  single unified stream (`thought -> text -> functionCall`) with tools
  attached nearly every turn; split-model routing or fallback delegation
  (e.g., to Gemini Flash) bypasses model reasoning and introduces severe
  latency.
