---
topic: slack-socket-mode
category: knowledge
tags: [knowledge, slack, socket-mode, mrkdwn, bolt]
updated_at: 2026-10-30T14:36:10.703208+00:00
compacted_at: 2026-10-30T14:36:10.703208+00:00
confidence: 0.95
---

# Knowledge: Slack Socket Mode, Threading & Mrkdwn

- Offload long-running turn dispatch and execution to a `ThreadPoolExecutor`
when handling Slack Bolt Socket Mode events to prevent starving the WebSocket
connection listener.

- Opening a direct message channel with a Slack bot triggers an app_home_opened
event; registering an @app.event("app_home_opened") listener prevents 404
unhandled request warnings in Slack Bolt.

- When reading thread history via Slack's conversations_replies API, use
cursor-based pagination (response_metadata.next_cursor) to handle threads larger
than 100 messages.
- Slack messages without plain text (such as attachments or Block Kit
components) need placeholder fallback handling when serializing thread history
into prompt context.

- Slack uses mrkdwn (*bold*, _italic_, ~strike~, <url|text>, • bullets) rather
than standard Markdown.
- Converting Markdown to Slack mrkdwn requires protecting code blocks and inline
code with unique sentinels and restoring placeholders in reverse order to
resolve nested formatting.

- Dynamic text inserted into Slack messages alongside URL links (<URL>) must
have '<', '>', and '&' characters sanitized to prevent breaking Slack link
syntax.
