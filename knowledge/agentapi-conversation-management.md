---
topic: agentapi-conversation-management
category: knowledge
tags: [knowledge, agentapi-conversation-management]
updated_at: 2026-09-13T00:20:29.621226+00:00
confidence: 0.95
---

# Knowledge: Agentapi-Conversation-Management

- agentapi new-conversation creates persistent directories under
~/.gemini/antigravity-cli/brain/<uuid> that get discovered as standard
conversations unless explicitly filtered or deleted.
- Sub-agent conversation transcripts can be programmatically detected by
checking message routing headers like 'Message from Root Agent'.

- `agentapi new-conversation` accepts an optional
`--model=<flash_lite|flash|pro>` flag to override the workspace's default
inherited model tier.
