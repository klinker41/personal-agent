---
topic: autobrr-api-scopes
category: knowledge
tags: [knowledge, autobrr-api-scopes]
updated_at: 2026-11-22T07:17:49.527226+00:00
confidence: 0.95
---

# Knowledge: Autobrr-Api-Scopes

- autobrr v1.91.0+ enforces granular API key permission scopes and migrates
existing keys in autobrr.db to hashed keys with default scopes
{"filters:write","lists:invoke"}, causing 403 Forbidden errors for read or
indexer integrations.
- Homepage's autobrr widget requires releases:read, filters:read, and
indexers:read scopes (for /api/release/stats and /api/release/indexers), while
Mousetrap requires indexers:read and indexers:write scopes to update MAM indexer
credentials.
- Modifying API key scopes directly in autobrr.db requires restarting the
autobrr container for the updated permissions to take effect.
