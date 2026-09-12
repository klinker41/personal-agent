---
topic: eventsource-stream-lifecycle
category: knowledge
tags: [knowledge, eventsource-stream-lifecycle]
updated_at: 2026-09-12T00:02:09.026888+00:00
confidence: 0.95
---

# Knowledge: Eventsource-Stream-Lifecycle

- Browser EventSource automatically retries connections infinitely on failure;
call eventSource.close() within the onerror handler to prevent persistent
reconnection loops on terminal or authorization errors.

- Explicitly calling es.close() within the EventSource onerror handler prevents
browser runaway reconnection loops upon stream failure.

- Fatal EventSource stream errors must explicitly call `es.close()` to prevent
native browser infinite reconnection storms.
