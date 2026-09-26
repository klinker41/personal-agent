---
topic: connectrpc-reverse-proxy
category: knowledge
tags: [knowledge, connectrpc-reverse-proxy]
updated_at: 2026-09-26T00:36:23.259152+00:00
confidence: 0.95
---

# Knowledge: Connectrpc-Reverse-Proxy

- **Traffic Interception & Model Injection**: Intercept Connect-RPC HTTP and
  WebSocket traffic, routing model calls to external providers while passing
  native traffic through. Augment `GetCascadeModelConfigData` responses to
  inject custom frontend model options.
- **Framing, Trailers & Buffering**: Avoid `.pipe()` (causes client hangs);
  invoke `res.flushHeaders()` immediately after `res.writeHead()` to prevent
  stalled streams. Strip hop-by-hop `Transfer-Encoding: chunked` to prevent
  double-chunking, and forward HTTP trailers (`TE: trailers` on requests;
  `Trailer` and `grpc-status: 0` on responses).
- **Socket Lifecycle & Disconnects**: Explicitly terminate upstream sockets on
  client disconnects to prevent orphaned connections and buffer bloat
  (Send-Q/Recv-Q). Avoid unconditionally destroying sockets on
  `res.on("close")` (triggers on both completions and disconnects) to prevent
  TCP RST packets from aborting active upstream contexts.
- **Protocol Translation & Tool Sanitization**: Map Anthropic streaming
  deltas to `agy` Protobuf events (`thinking_delta` to the UI thinking drawer;
  `tool_use` and `input_json_delta` to `GetChatMessageResponse` frames). When
  proxying tool calls to external providers (OpenAI, Anthropic), sanitize
  arguments by coercing stringified booleans and numbers to native types,
  stripping `ArtifactMetadata` outside the brain directory to prevent schema
  validation errors, and supplying defaults for artifact files.
