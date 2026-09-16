---
topic: connectrpc-reverse-proxy
category: knowledge
tags: [knowledge, connectrpc-reverse-proxy]
updated_at: 2026-09-16T00:39:41.498191+00:00
confidence: 0.95
---

# Knowledge: Connectrpc-Reverse-Proxy

- **Traffic Interception & Model Injection**: Intercept Connect-RPC traffic
  (HTTP and WebSocket) to selectively route model requests to alternate
  providers while passing native traffic through. Augment responses to
  `GetCascadeModelConfigData` to inject custom frontend model options.
- **HTTP Trailers, Framing & Buffering**: Forward HTTP trailers
  (`TE: trailers` on requests; `Trailer` and `grpc-status: 0` on responses);
  omitting them or using `.pipe()` hangs clients indefinitely. Call
  `res.flushHeaders()` immediately after `res.writeHead()` to avoid stalled
  streams. Strip hop-by-hop `Transfer-Encoding: chunked` before
  `res.writeHead()` to prevent double-chunking and framing errors.
- **Socket Lifecycle & Disconnects**: Explicitly close upstream sockets on
  client disconnect to prevent orphaned connections from bloating TCP buffers
  (Send-Q/Recv-Q). Never unconditionally destroy sockets on `res.on("close")`,
  which fires on normal completion as well as disconnects, emitting TCP RST
  packets that prematurely cancel upstream contexts.
- **Protocol Translation & Tool Sanitization**: Anthropic streaming deltas
  map to `agy` Protobuf events: `thinking_delta` routes to the UI thinking
  drawer, while `tool_use` and `input_json_delta` map to
  `GetChatMessageResponse` tool call frames. When proxying tool calls to
  external LLMs (OpenAI, Anthropic), sanitize arguments: coerce
  string-serialized booleans and numbers to native types, strip
  `ArtifactMetadata` outside the brain directory to prevent schema
  validation failures, and supply defaults for artifact files.
