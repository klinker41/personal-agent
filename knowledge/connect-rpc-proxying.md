---
topic: connect-rpc-proxying
category: knowledge
tags: [knowledge, connect-rpc-proxying]
updated_at: 2026-08-29T11:54:13.487343+00:00
confidence: 0.95
---

# Knowledge: Connect-Rpc-Proxying

- Connect-RPC and gRPC-Web require preserving 'TE: trailers' from requests and
'Trailer' headers from responses; stripping them as hop-by-hop headers breaks
end-of-stream signaling.
- In Node.js HTTP proxies, 'res.on("close")' fires on both normal completion and
premature disconnects; unconditionally destroying upstream sockets on close
sends TCP RST packets that cancel server-side request contexts.
- Node.js buffers 'res.writeHead()' headers until body chunks arrive unless
'res.flushHeaders()' is called, which can stall browser streaming readers on
long-lived RPC streams.
