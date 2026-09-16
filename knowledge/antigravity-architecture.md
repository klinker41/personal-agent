---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-09-16T00:39:21.278322+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- **Daemon & Web UI IPC**: The Antigravity Web UI communicates via local
  RPC/WebSockets with the compiled local `agy` daemon, which executes tools and
  orchestrates backend model requests rather than invoking them from the
  browser.
- **Connect-RPC Configuration**: The internal Connect-RPC endpoint
  `SetCloudCodeURL` strictly requires parameter key `url` (not `cloudCodeUrl`).
  Mismatches cause Go Protobuf unmarshaling to default to an empty string,
  failing with
  `Post "/v1internal:loadCodeAssist": unsupported protocol scheme ""` and
  crashing the session until container restart.
- **Cloud Code Protocol & SSE Translation**: `agy` targets internal Cloud Code
  PA endpoints (`/v1internal:streamGenerateContent`,
  `/v1internal:fetchAvailableModels`) using HTTP/2 Protobuf
  (`JetskiService`/`GetChatMessageRequest`). It expects streaming SSE chunks
  wrapped in `{"response": {"candidates": [...]}}` rather than standard Gemini
  AI Studio payloads, requiring a bidirectional translation proxy for external
  LLMs (Anthropic, OpenAI).
- **Model ID Placeholder Routing**: Built-in models use placeholders in the
  M500–M649 range (e.g., M599, M605), overlapping custom model ranges. Reverse
  proxies must pass unregistered placeholders in this range through to Google
  Cloud Code rather than falling back to default custom model sessions.
- **Streaming Flow & Single-Model Routing**: Antigravity attaches tool
  declarations to nearly every turn and expects a unified streaming sequence
  (`thought -> text -> functionCall`). Multi-model split routing or delegating
  tool-bearing prompts to fallback models (e.g., Gemini Flash) bypasses custom
  model reasoning and introduces severe latency.
- **Proxy Tool Argument Sanitization**: Because third-party LLMs (Claude, GPT)
  routinely omit required parameters, the translation proxy must sanitize tool
  arguments. Specifically, `write_to_file` rejects overwriting existing files
  unless `Overwrite: true` is explicitly passed and requires `UserFacing` when
  `ArtifactMetadata` is provided.
