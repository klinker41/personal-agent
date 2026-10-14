---
topic: eventsource-stream-lifecycle
category: knowledge
tags: [knowledge, eventsource-stream-lifecycle]
updated_at: 2026-10-14T00:02:26.908000+00:00
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

- Server-Sent Events (SSE) streams require periodic heartbeat or comment pings
(e.g., ': ping\n\n') to keep connections alive through intermediate reverse
proxies and prevent premature timeouts.

- Send periodic comment frames (such as ': ping\n\n') as SSE heartbeats to
prevent proxies and browsers from timing out without triggering client-side
message listeners.
