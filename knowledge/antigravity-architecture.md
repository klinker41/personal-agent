---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-09-14T00:37:39.867232+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- The Antigravity Web UI communicates via local RPC/WebSockets with the
  compiled local `agy` daemon, which directly executes tools and manages
  backend model requests rather than making calls from the browser.
- The `agy` daemon targets internal Cloud Code PA endpoints
  (`/v1internal:streamGenerateContent`, `/v1internal:fetchAvailableModels`)
  using HTTP/2 Protobuf (`JetskiService`/`GetChatMessageRequest`). It expects
  streaming SSE chunks wrapped in an outer structure
  (`{"response": {"candidates": [...]}}`) rather than standard Gemini AI
  Studio payloads, requiring a bidirectional translation proxy for external
  LLMs (Anthropic, OpenAI).
- The internal Connect-RPC endpoint `SetCloudCodeURL` expects the parameter
  key `url` rather than `cloudCodeUrl`. Mismatched keys cause Go Protobuf
  unmarshaling to default to an empty string, breaking endpoints with
  `Post "/v1internal:loadCodeAssist": unsupported protocol scheme ""` and
  crashing the session until container restart.
- Built-in models use placeholders in the M500-M649 range (e.g., M599, M605),
  which overlap with custom model ranges. Reverse proxies must ensure
  unregistered placeholders in this range fall through to Google Cloud Code
  instead of falling back to default custom model sessions.
- Antigravity attaches tool declarations to nearly every turn and expects a
  unified streaming sequence (`thought -> text -> functionCall`). Multi-model
  split routing or delegating tool-bearing prompts to a fallback model (e.g.,
  Gemini Flash) bypasses custom models for agentic reasoning and introduces
  severe protocol complexity and latency.
- The `write_to_file` tool rejects overwriting existing files unless
  `Overwrite: true` is explicitly passed, and requires `UserFacing` whenever
  `ArtifactMetadata` is provided. Third-party models (Claude, GPT) routinely
  omit `Overwrite`, requiring proxy-level argument sanitization.
