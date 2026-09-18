---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-09-18T00:38:15.227306+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- **Daemon IPC & Connect-RPC**: The Web UI interacts via local RPC and
  WebSockets with the compiled `agy` daemon, which coordinates tools and model
  requests. The internal Connect-RPC `SetCloudCodeURL` endpoint strictly
  requires parameter `url` (never `cloudCodeUrl`); unmarshaling mismatches
  default to empty, crashing the session until a container restart with
  `Post "/v1internal:loadCodeAssist": unsupported protocol scheme ""`.
- **Cloud Code Protocol & Translation Proxy**: `agy` calls internal Cloud Code
  PA endpoints (`/v1internal:streamGenerateContent` and
  `/v1internal:fetchAvailableModels`) via HTTP/2 Protobuf
  (`JetskiService`/`GetChatMessageRequest`). Streaming SSE chunks must be
  wrapped in `{"response": {"candidates": [...]}}` rather than standard Gemini
  AI Studio payloads, requiring a bidirectional translation proxy for external
  LLMs (OpenAI, Anthropic).
- **Model Routing & Stream Execution**: Built-in models use placeholder IDs in
  the M500–M649 range (e.g., M599, M605). Proxies must route unregistered IDs
  in this range to Cloud Code rather than custom sessions. `agy` expects a
  single unified stream (`thought -> text -> functionCall`) with tools attached
  nearly every turn; split-model routing or fallback delegation (e.g., to
  Gemini Flash) bypasses model reasoning and adds severe latency.
- **Proxy Tool Argument Sanitization**: Because external LLMs often omit
  required fields, the translation proxy must sanitize tool payloads:
  `write_to_file` requires `Overwrite: true` to overwrite files and mandates
  `UserFacing` whenever `ArtifactMetadata` is provided.
