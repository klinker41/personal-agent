---
topic: slack-mrkdwn-conversion
category: knowledge
tags: [knowledge, slack-mrkdwn-conversion]
updated_at: 2026-08-29T11:54:29.040400+00:00
confidence: 0.95
---

# Knowledge: Slack-Mrkdwn-Conversion

- Slack uses mrkdwn (*bold*, _italic_, ~strike~, <url|text>, • bullets) rather
than standard Markdown.
- Converting Markdown to Slack mrkdwn requires protecting code blocks and inline
code with unique sentinels and restoring placeholders in reverse order to
resolve nested formatting.
