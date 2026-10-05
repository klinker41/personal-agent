---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-10-05T00:38:20.538149+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- **Daemon IPC & Hub State**: The Web UI communicates via local RPC and
  WebSockets with the compiled `agy` daemon. Connect-RPC endpoint
  `SetCloudCodeURL` strictly requires parameter `url` (never `cloudCodeUrl`);
  mismatches default to empty, crashing the session until a container restart
  with `Post "/v1internal:loadCodeAssist": unsupported protocol scheme ""`.
  The hub's `StreamAgentStateUpdates` stream yields semantic plain JSON
  snapshots of conversation steps with incrementally growing thinking text,
  eliminating the need to decode binary protobuf field-number diff streams.
- **Cloud Code Protocol & Translation Proxy**: `agy` calls internal Cloud
  Code endpoints (`/v1internal:streamGenerateContent` and
  `/v1internal:fetchAvailableModels`) via HTTP/2 Protobuf
  (`JetskiService`/`GetChatMessageRequest`). Integrating external LLMs
  requires a bidirectional translation proxy that wraps streaming SSE chunks
  in `{"response": {"candidates": [...]}}` (not standard Gemini AI Studio
  payloads) and sanitizes arguments when models omit required fields (e.g.,
  enforcing `Overwrite: true` and mandating `UserFacing` in `ArtifactMetadata`
  for `write_to_file`).
- **Model Routing & Stream Execution**: Placeholder IDs in the M500–M649
  range (e.g., M599, M605) represent built-in models; proxies must route
  unregistered IDs in this range to Cloud Code instead of custom sessions.
  `agy` requires a single unified stream (`thought -> text -> functionCall`)
  with tools attached nearly every turn; split-model routing or fallback
  delegation (e.g., to Gemini Flash) bypasses model reasoning and introduces
  severe latency.
