---
topic: antigravity-architecture
category: knowledge
tags: [knowledge, antigravity-architecture]
updated_at: 2026-09-13T00:20:21.480535+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Architecture

- Antigravity Web UI communicates via local RPC/WebSockets to the compiled local
agy daemon, which directly executes tools and sends outbound HTTPS requests to
Gemini API servers rather than making model calls directly from the browser.

- The `agy` internal Connect-RPC endpoint `SetCloudCodeURL` expects the field
name `url` rather than `cloudCodeUrl`; sending an incorrect key causes Go
Protobuf unmarshaling to set an empty string, breaking CloudCode endpoints
(`Post "/v1internal:loadCodeAssist": unsupported protocol scheme ""`) and
crashing the agent session until container restart.
- `agy` communicates with the Google Cloud backend via internal Protobuf over
HTTP/2 (`JetskiService`/`GetChatMessageRequest`), requiring a bidirectional
translation proxy to adapt external LLM APIs (Anthropic, OpenAI) to the
Antigravity agent loop.

- Antigravity built-in models use placeholder identifiers in the M500-M649 range
(e.g. M599, M605), overlapping with custom model placeholders; reverse proxies
must verify unregistered placeholders in this range fall through to Google Cloud
Code rather than falling back to default custom model sessions.
- Antigravity's write_to_file tool rejects overwrites unless Overwrite: true is
explicitly provided; third-party models (e.g., Claude, GPT) frequently omit this
parameter on existing files, requiring proxy-level argument sanitization.
