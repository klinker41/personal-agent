---
topic: connectrpc-reverse-proxy
category: knowledge
tags: [knowledge, connectrpc-reverse-proxy]
updated_at: 2026-09-21T00:35:54.922848+00:00
confidence: 0.95
---

# Knowledge: Connectrpc-Reverse-Proxy

- **Traffic Interception & Model Injection**: Intercept Connect-RPC HTTP and
  WebSocket traffic to route model requests to alternate providers while
  passing native traffic through. Augment `GetCascadeModelConfigData`
  responses to inject custom frontend model options.
- **HTTP Framing, Trailers & Buffering**: Forward HTTP trailers
  (`TE: trailers` on requests; `Trailer` and `grpc-status: 0` on responses);
  omitting them or using `.pipe()` hangs clients indefinitely. Strip
  hop-by-hop `Transfer-Encoding: chunked` to prevent double-chunking errors,
  and call `res.flushHeaders()` immediately after `res.writeHead()` to avoid
  stalled streams.
- **Socket Lifecycle & Disconnects**: Explicitly close upstream sockets on
  client disconnect to prevent orphaned connections from bloating TCP buffers
  (Send-Q/Recv-Q). Never destroy sockets unconditionally on
  `res.on("close")` (which fires on both normal completion and disconnects)
  to prevent TCP RST packets from prematurely canceling upstream contexts.
- **Protocol Translation & Tool Sanitization**: Map Anthropic streaming
  deltas to `agy` Protobuf events (`thinking_delta` to the UI thinking drawer;
  `tool_use` and `input_json_delta` to `GetChatMessageResponse` tool call
  frames). When proxying tool calls to external LLMs (OpenAI, Anthropic),
  sanitize arguments by coercing stringified booleans and numbers to native
  types, stripping `ArtifactMetadata` outside the brain directory to prevent
  schema validation errors, and supplying defaults for artifact files.
