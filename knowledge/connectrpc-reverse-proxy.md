---
topic: connectrpc-reverse-proxy
category: knowledge
tags: [knowledge, connectrpc-reverse-proxy]
updated_at: 2026-09-19T00:38:04.302272+00:00
confidence: 0.95
---

# Knowledge: Connectrpc-Reverse-Proxy

- **Traffic Interception & Model Injection**: Intercept Connect-RPC HTTP and
  WebSocket traffic to route model requests to alternate providers while
  passing native traffic through. Augment `GetCascadeModelConfigData` responses
  to inject custom frontend model options.
- **HTTP Trailers, Framing & Buffering**: Forward HTTP trailers (`TE: trailers`
  on requests; `Trailer` and `grpc-status: 0` on responses); omitting them or
  using `.pipe()` hangs clients indefinitely. Call `res.flushHeaders()`
  immediately after `res.writeHead()` to prevent stalled streams, and strip
  hop-by-hop `Transfer-Encoding: chunked` beforehand to avoid double-chunking
  and framing errors.
- **Socket Lifecycle & Disconnect Handling**: Explicitly close upstream sockets
  on client disconnect to prevent orphaned connections from bloating TCP
  buffers (Send-Q/Recv-Q). Never unconditionally destroy sockets on
  `res.on("close")` (which fires on normal completion and disconnects alike)
  to avoid emitting TCP RST packets that prematurely cancel upstream contexts.
- **Protocol Translation & Tool Sanitization**: Map Anthropic streaming deltas
  to `agy` Protobuf events (`thinking_delta` to the UI thinking drawer;
  `tool_use` and `input_json_delta` to `GetChatMessageResponse` tool call
  frames). When proxying tool calls to external LLMs (OpenAI, Anthropic),
  sanitize arguments by coercing stringified booleans/numbers to native types,
  stripping `ArtifactMetadata` outside the brain directory to prevent schema
  validation errors, and supplying defaults for artifact files.
