---
topic: connectrpc-reverse-proxy
category: knowledge
tags: [knowledge, connectrpc-reverse-proxy]
updated_at: 2026-08-29T11:58:09.902540+00:00
confidence: 0.95
---

# Knowledge: Connectrpc-Reverse-Proxy

- Connect-RPC and gRPC-Web stream completion requires preserving and forwarding
  HTTP trailers (such as `TE: trailers` from requests and `Trailer` headers /
  `grpc-status: 0` from responses); stripping them as hop-by-hop headers or
  using standard `.pipe()` operations that drop trailers causes clients to hang
  indefinitely.
- Reverse proxies proxying ConnectRPC / gRPC-Web streams must explicitly close
  upstream sockets on client disconnect to prevent orphaned connections from
  filling TCP buffers (Send-Q/Recv-Q) and blocking updates; however, in Node.js
  HTTP proxies, `res.on("close")` fires on both normal completion and premature
  disconnects, so unconditionally destroying sockets on close sends TCP RST
  packets that cancel server-side request contexts.
- Node.js buffers `res.writeHead()` headers until body chunks arrive unless
  `res.flushHeaders()` is called, which can stall browser streaming readers on
  long-lived RPC streams.
- Forwarding hop-by-hop `Transfer-Encoding: chunked` headers into
  `res.writeHead()` causes double-chunking and framing errors on streaming
  responses.
