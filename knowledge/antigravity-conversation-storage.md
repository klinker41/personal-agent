---
topic: antigravity-conversation-storage
category: knowledge
tags: [knowledge, antigravity-conversation-storage]
updated_at: 2026-08-30T11:29:35.180226+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Conversation-Storage

- Antigravity stores conversations in SQLite (.db, -wal, -shm); errors in
cli.log about missing .pb files are fallback errors when SQLite databases or
state files fail to load.
- Overwriting 'antigravity_state.pbtxt' resets installation_uuid and schema
migration markers, causing Antigravity to treat the instance as a fresh
installation and break trajectory loading.
- Conversations are project-scoped and automatically archived; accessing
existing chats on new client sessions requires selecting the active project
workspace and checking the archived filter.

- Antigravity stores conversation trajectories locally on the filesystem
(~/.gemini/antigravity-cli/conversations/<id>.db and
~/.gemini/antigravity-cli/brain/<id>/), scoped by project UUID and workspace
URI.

- Antigravity persists conversations across three distinct locations on disk:
transcripts/artifacts in ~/.gemini/antigravity-cli/brain/<id>/, SQLite records
in ~/.gemini/antigravity-cli/conversations/<id>.db, and sidebar titles in
~/.gemini/antigravity-cli/annotations/<id>.pbtxt. Full deletion requires purging
all three.
