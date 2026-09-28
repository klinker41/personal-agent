---
topic: connectrpc-reverse-proxy
category: knowledge
tags: [knowledge, connectrpc-reverse-proxy]
updated_at: 2026-09-28T00:34:56.103994+00:00
confidence: 0.95
---

# Knowledge: Connectrpc-Reverse-Proxy

- **Traffic Interception & Model Injection**: Intercept Connect-RPC HTTP and
  WebSocket streams to route model calls externally while passing native
  traffic through. Augment `GetCascadeModelConfigData` responses to inject
  custom frontend model options.
- **Framing & Trailer Management**: Avoid `.pipe()` (causes client hangs).
  Call `res.flushHeaders()` immediately after `res.writeHead()` to prevent
  stalled streams. Strip hop-by-hop `Transfer-Encoding: chunked` to prevent
  double-chunking, and forward HTTP trailers (`TE: trailers` on requests;
  `Trailer` and `grpc-status: 0` on responses).
- **Socket Lifecycle & Cleanup**: Terminate upstream sockets on client
  disconnects to avoid orphaned connections and Send-Q/Recv-Q buffer bloat.
  Do not unconditionally destroy sockets on `res.on("close")` (which fires on
  both completions and disconnects) to prevent TCP RST packets from aborting
  active upstream contexts.
- **Protocol Translation & Tool Sanitization**: Map Anthropic streaming
  deltas to `agy` Protobuf events (`thinking_delta` to the UI thinking drawer;
  `tool_use` and `input_json_delta` to `GetChatMessageResponse` frames).
  Sanitize tool arguments for external providers by coercing stringified
  booleans/numbers to native types, stripping `ArtifactMetadata` outside the
  brain directory to avoid validation errors, and supplying defaults for
  artifact files.
