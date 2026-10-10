---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-10-10T00:39:20.115294+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- **Daemon IPC & Hub State**: The Web UI communicates with `agy` via
  Connect-RPC and WebSockets. RPC `SetCloudCodeURL` strictly requires
  parameter `url` (not `cloudCodeUrl`); mismatches leave it empty and
  crash the session (`Post "/v1internal:loadCodeAssist": unsupported protocol
  scheme ""`) until the container is restarted. Hub stream
  `StreamAgentStateUpdates` emits plain JSON step snapshots with incremental
  thinking text, avoiding binary protobuf diff decoding.
- **Cloud Code Protocol & Translation Proxy**: `agy` connects to internal
  Cloud Code endpoints (`/v1internal:streamGenerateContent` and
  `/v1internal:fetchAvailableModels`) via HTTP/2 Protobuf
  (`JetskiService`/`GetChatMessageRequest`). Integrating external LLMs
  requires a bidirectional proxy transforming streaming SSE chunks into
  `{"response": {"candidates": [...]}}` (not Gemini AI Studio schemas) and
  injecting missing required tool parameters (e.g., enforcing
  `Overwrite: true` and `ArtifactMetadata.UserFacing` for `write_to_file`).
- **Model Routing & Stream Execution**: Placeholder IDs in range M500–M649
  (e.g., M599, M605) denote built-in models; proxies must route unregistered
  IDs in this range directly to Cloud Code. `agy` enforces a single unified
  stream (`thought -> text -> functionCall`) with tools attached almost every
  turn; split-model routing or fallback delegation (e.g., to Gemini Flash)
  bypasses reasoning and introduces severe latency.
