---
topic: asyncio-stream-backpressure
category: knowledge
tags: [knowledge, asyncio-stream-backpressure]
updated_at: 2026-09-14T00:16:49.191945+00:00
confidence: 0.95
---

# Knowledge: Asyncio-Stream-Backpressure

- Calling await writer.drain() directly inside a locked execution loop creates
severe deadlocks if any consumer stalls; Linux socket buffers (128KB–212KB) can
saturate within 2 frames of high-throughput media.
- Prevent slow-client broadcast stalls by routing socket I/O through per-client
bounded queues with a drop-oldest eviction policy on overflow, handled by
dedicated sender tasks with drain timeouts.
