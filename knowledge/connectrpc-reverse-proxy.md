---
topic: connectrpc-reverse-proxy
category: knowledge
tags: [knowledge, connectrpc-reverse-proxy]
updated_at: 2026-08-29T11:58:09.902540+00:00
confidence: 0.95
---

# Knowledge: Connectrpc-Reverse-Proxy

- Reverse proxies proxying ConnectRPC / gRPC-Web streams must explicitly close
upstream sockets on client disconnect; orphaned streaming connections can fill
TCP buffers (Send-Q/Recv-Q) and block reactive updates.
- Forwarding hop-by-hop `Transfer-Encoding: chunked` headers into
`res.writeHead()` causes double-chunking and framing errors on streaming
responses.
- ConnectRPC stream completion requires forwarding HTTP trailers (such as
`grpc-status: 0`); standard `.pipe()` operations that drop trailers cause
clients to hang indefinitely.
