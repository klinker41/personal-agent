---
topic: slack-conversations-replies
category: knowledge
tags: [knowledge, slack-conversations-replies]
updated_at: 2026-08-29T11:55:12.803750+00:00
confidence: 0.95
---

# Knowledge: Slack-Conversations-Replies

- When reading thread history via Slack's conversations_replies API, use
cursor-based pagination (response_metadata.next_cursor) to handle threads larger
than 100 messages.
- Slack messages without plain text (such as attachments or Block Kit
components) need placeholder fallback handling when serializing thread history
into prompt context.
