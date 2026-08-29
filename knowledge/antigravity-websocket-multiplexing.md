---
topic: antigravity-websocket-multiplexing
category: knowledge
tags: [knowledge, antigravity-websocket-multiplexing]
updated_at: 2026-08-29T11:55:32.324902+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Websocket-Multiplexing

- The Antigravity web UI opens 5+ concurrent persistent server-streaming RPC
connections, easily hitting browser HTTP/1.1 6-connection per-origin limits and
stalling pending requests (e.g., conversation history and prompt submissions).
- Appending 'useWebSocket=true' to the Antigravity URL activates built-in
WebSocket RPC multiplexing over '/connect-websocket', routing all streaming and
unary calls through a single WebSocket connection.
