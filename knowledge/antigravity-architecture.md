---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-09-17T00:37:52.414291+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- **Daemon IPC & Connect-RPC**: Web UI communicates via local RPC/WebSockets
  with the compiled `agy` daemon, which executes tools and orchestrates backend
  model requests. Internal Connect-RPC endpoint `SetCloudCodeURL` strictly
  requires parameter key `url` (not `cloudCodeUrl`); mismatches cause Go
  Protobuf unmarshaling to default to empty, failing with
  `Post "/v1internal:loadCodeAssist": unsupported protocol scheme ""` and
  crashing the session until container restart.
- **Cloud Code Protocol & SSE Translation**: `agy` targets internal Cloud Code
  PA endpoints (`/v1internal:streamGenerateContent` and
  `/v1internal:fetchAvailableModels`) using HTTP/2 Protobuf
  (`JetskiService`/`GetChatMessageRequest`). It expects streaming SSE chunks
  wrapped in `{"response": {"candidates": [...]}}` rather than standard Gemini
  AI Studio payloads, requiring a bidirectional translation proxy for external
  LLMs (Anthropic, OpenAI).
- **Model Routing & Streaming Flow**: Built-in models use placeholders in the
  M500–M649 range (e.g., M599, M605), overlapping custom ranges; proxies must
  pass unregistered IDs in this range to Cloud Code instead of default custom
  sessions. Furthermore, `agy` expects a single unified stream
  (`thought -> text -> functionCall`) with tools attached to nearly every turn;
  multi-model split routing or fallback delegation (e.g., to Gemini Flash)
  bypasses custom model reasoning and introduces severe latency.
- **Proxy Tool Argument Sanitization**: Because third-party LLMs (Claude, GPT)
  routinely omit required parameters, the translation proxy must sanitize tool
  arguments: `write_to_file` rejects file overwrites without explicit
  `Overwrite: true` and requires `UserFacing` when `ArtifactMetadata` is given.
