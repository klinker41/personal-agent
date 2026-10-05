---
topic: antigravity-websocket-multiplexing
category: knowledge
tags: [knowledge, antigravity-websocket-multiplexing]
updated_at: 2026-10-05T00:01:04.486555+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Websocket-Multiplexing

- The Antigravity web UI opens 5+ concurrent persistent server-streaming RPC
connections, easily hitting browser HTTP/1.1 6-connection per-origin limits and
stalling pending requests (e.g., conversation history and prompt submissions).
- Appending 'useWebSocket=true' to the Antigravity URL activates built-in
WebSocket RPC multiplexing over '/connect-websocket', routing all streaming and
unary calls through a single WebSocket connection.

- Mobile gateway stream architecture multiplexes multiple conversation streams
and list updates over a single device WebSocket connection using per-topic
sequence numbers and a replay buffer to handle network reconnects.
