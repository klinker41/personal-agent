---
topic: connectrpc-reverse-proxy
category: knowledge
tags: [knowledge, connectrpc-reverse-proxy]
updated_at: 2026-09-13T00:34:41.671531+00:00
confidence: 0.95
---

# Knowledge: Connectrpc-Reverse-Proxy

- **HTTP Trailers & Framing**: Preserving and forwarding HTTP trailers
  (such as `TE: trailers` on requests and `Trailer` / `grpc-status: 0` on
  responses) is required; stripping them or using standard `.pipe()` hangs
  clients indefinitely. Avoid forwarding hop-by-hop
  `Transfer-Encoding: chunked` into `res.writeHead()` to prevent
  double-chunking and framing errors.
- **Node.js Buffering & Sockets**: Always call `res.flushHeaders()` after
  `res.writeHead()` so browser readers do not stall on buffered headers.
  Explicitly close upstream sockets on client disconnect to prevent orphaned
  connections from bloating TCP buffers (Send-Q/Recv-Q). Avoid unconditionally
  destroying sockets on `res.on("close")`, which fires on normal completion as
  well as disconnects, sending TCP RST packets that cancel upstream contexts.
- **Protocol Mapping & Tool Sanitization**: Anthropic streaming deltas map
  cleanly to `agy` Protobuf events: `thinking_delta` routes to the UI
  thinking drawer, while `tool_use` and `input_json_delta` map to
  `GetChatMessageResponse` tool call frames. When proxying tool calls to
  third-party LLMs (OpenAI, Anthropic), sanitize arguments: coerce
  string-serialized booleans and numbers to native types, strip
  `ArtifactMetadata` on paths outside the brain directory to prevent schema
  validation failures, and supply defaults for artifact files.
- **Frontend Model Injection**: Custom model options can be natively injected
  into the Antigravity frontend dropdown by intercepting and augmenting the
  `GetCascadeModelConfigData` Connect-RPC response.
