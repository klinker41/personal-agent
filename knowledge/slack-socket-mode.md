---
topic: slack-socket-mode
category: knowledge
tags: [knowledge, slack-socket-mode]
updated_at: 2026-08-29T11:53:44.813877+00:00
confidence: 0.95
---

# Knowledge: Slack-Socket-Mode

- Offload long-running turn dispatch and execution to a `ThreadPoolExecutor`
when handling Slack Bolt Socket Mode events to prevent starving the WebSocket
connection listener.
