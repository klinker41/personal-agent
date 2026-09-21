---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-09-21T00:35:42.773424+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- **Daemon IPC & Connect-RPC**: The Web UI interacts via local RPC and
  WebSockets with the compiled `agy` daemon to coordinate tools and model
  requests. Connect-RPC endpoint `SetCloudCodeURL` strictly requires parameter
  `url` (never `cloudCodeUrl`); unmarshaling mismatches default to empty,
  crashing the session until a container restart with
  `Post "/v1internal:loadCodeAssist": unsupported protocol scheme ""`.
- **Cloud Code Protocol & Translation Proxy**: `agy` calls internal Cloud Code
  PA endpoints (`/v1internal:streamGenerateContent` and
  `/v1internal:fetchAvailableModels`) via HTTP/2 Protobuf
  (`JetskiService`/`GetChatMessageRequest`). Integrating external LLMs (OpenAI,
  Anthropic) requires a bidirectional translation proxy that wraps streaming
  SSE chunks in `{"response": {"candidates": [...]}}` (not standard Gemini AI
  Studio payloads) and sanitizes arguments when models omit required fields
  (e.g., enforcing `Overwrite: true` to overwrite files and mandating
  `UserFacing` in `ArtifactMetadata` for `write_to_file`).
- **Model Routing & Stream Execution**: Built-in models use placeholder IDs in
  the M500–M649 range (e.g., M599, M605); proxies must route unregistered IDs
  in this range to Cloud Code instead of custom sessions. `agy` requires a
  single unified stream (`thought -> text -> functionCall`) with tools attached
  nearly every turn; split-model routing or fallback delegation (e.g., to
  Gemini Flash) bypasses model reasoning and introduces severe latency.
