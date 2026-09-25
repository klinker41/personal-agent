---
topic: connectrpc-reverse-proxy
category: knowledge
tags: [knowledge, connectrpc-reverse-proxy]
updated_at: 2026-09-25T00:36:42.916873+00:00
confidence: 0.95
---

# Knowledge: Connectrpc-Reverse-Proxy

- **Traffic Interception & Model Injection**: Intercept Connect-RPC HTTP and
  WebSocket traffic, routing model calls to external providers while passing
  native traffic through. Augment `GetCascadeModelConfigData` responses to
  inject custom frontend model options.
- **HTTP Framing, Trailers & Buffering**: Forward HTTP trailers
  (`TE: trailers` on requests; `Trailer` and `grpc-status: 0` on responses)
  and avoid `.pipe()`, which hangs clients indefinitely. Strip hop-by-hop
  `Transfer-Encoding: chunked` to prevent double-chunking, and invoke
  `res.flushHeaders()` immediately after `res.writeHead()` to prevent stalled
  streams.
- **Socket Lifecycle & Disconnects**: Explicitly terminate upstream sockets on
  client disconnects to prevent orphaned connections and bloated TCP buffers
  (Send-Q/Recv-Q). Avoid unconditionally destroying sockets on
  `res.on("close")` (triggers on both completions and disconnects) to prevent
  TCP RST packets from aborting active upstream contexts.
- **Protocol Translation & Tool Sanitization**: Map Anthropic streaming deltas
  to `agy` Protobuf events (`thinking_delta` to the UI thinking drawer;
  `tool_use` and `input_json_delta` to `GetChatMessageResponse` frames). When
  proxying tool calls to external providers (OpenAI, Anthropic), sanitize
  arguments by coercing stringified booleans and numbers to native types,
  stripping `ArtifactMetadata` outside the brain directory to avoid schema
  validation errors, and providing defaults for artifact files.
