---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-09-15T00:38:07.934703+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- **Daemon & UI IPC**: The Antigravity Web UI communicates via local
  RPC/WebSockets with the compiled local `agy` daemon, which executes tools
  and orchestrates backend model requests rather than calling from the
  browser.
- **Cloud Code Protocol & Proxy Translation**: `agy` targets internal Cloud
  Code PA endpoints (`/v1internal:streamGenerateContent`,
  `/v1internal:fetchAvailableModels`) using HTTP/2 Protobuf
  (`JetskiService`/`GetChatMessageRequest`). It expects streaming SSE chunks
  wrapped in `{"response": {"candidates": [...]}}` rather than standard Gemini
  AI Studio payloads, requiring a bidirectional translation proxy for
  external LLMs (Anthropic, OpenAI).
- **Connect-RPC Configuration**: The internal Connect-RPC endpoint
  `SetCloudCodeURL` expects the parameter key `url` rather than `cloudCodeUrl`.
  Mismatches cause Go Protobuf unmarshaling to default to an empty string,
  failing with
  `Post "/v1internal:loadCodeAssist": unsupported protocol scheme ""` and
  crashing the session until container restart.
- **Model ID Placeholder Routing**: Built-in models use placeholders in the
  M500–M649 range (e.g., M599, M605), overlapping custom model ranges. Reverse
  proxies must pass unregistered placeholders in this range through to Google
  Cloud Code rather than falling back to default custom model sessions.
- **Turn Flow & Proxy Argument Sanitization**: Antigravity attaches tool
  declarations to nearly every turn and expects a unified streaming sequence
  (`thought -> text -> functionCall`). Multi-model split routing or delegating
  tool-bearing prompts to fallback models (e.g., Gemini Flash) bypasses custom
  model reasoning and adds severe latency. Additionally, `write_to_file`
  rejects overwriting existing files unless `Overwrite: true` is explicitly
  passed and requires `UserFacing` when `ArtifactMetadata` is provided; because
  third-party models (Claude, GPT) routinely omit `Overwrite`, the proxy must
  sanitize tool arguments.
