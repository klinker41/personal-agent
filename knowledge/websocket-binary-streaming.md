---
topic: websocket-binary-streaming
category: knowledge
tags: [knowledge, websocket-binary-streaming]
updated_at: 2026-09-14T00:39:55.747261+00:00
confidence: 0.95
---

# Knowledge: Websocket-Binary-Streaming

- When streaming mixed binary video and audio over WebSockets, maintain an
  isolated cache of the latest video frame on the streaming hub to immediately
  bootstrap newly connected clients and eliminate initial black screens without
  sending interleaved audio chunks.
- Trap individual socket exceptions during broadcast loops so disconnected or
  failing clients are pruned without disrupting transmissions to healthy
  subscribers.
- When parsing binary packet headers using DataView across sliced buffers or IPC
  streams, always initialize DataView with `buffer.byteOffset` and
  `buffer.byteLength` to prevent misalignment on sub-sliced `Uint8Array`
  buffers.
- Sequential `writer.drain()` calls across stream writers in a broadcast loop
  cause head-of-line blocking and freeze the event loop. In binary IPC and media
  broadcast loops, use per-client bounded queues (such as asyncio queues) with
  drop-oldest eviction and isolated sender loops with drain timeouts to prevent
  backpressure deadlocks, unbounded memory growth, and slow consumers blocking
  critical execution locks.
- Hono with Bun native WebSockets (`createBunWebSocket`) allows multiplexing
  binary frame streaming (such as RGBA or JPEG packets) alongside structured
  JSON state updates over a single WebSocket connection.
