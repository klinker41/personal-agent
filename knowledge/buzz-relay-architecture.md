---
topic: buzz-relay-architecture
category: knowledge
tags: [knowledge, buzz, acp, nostr, relay]
updated_at: 2026-10-30T14:36:10.703208+00:00
compacted_at: 2026-10-30T14:36:10.703208+00:00
confidence: 0.95
---

# Knowledge: Buzz Relay & ACP Adapter Architecture

- Block's Buzz is a Nostr-based relay implemented in Rust with Axum, requiring
PostgreSQL 17 for persistent storage, Redis 7 for pub/sub and presence
heartbeats, and S3/MinIO for Blossom protocol media storage.

- Buzz v0.5.0+ supports external agent harnesses generically via Agent Client
Protocol (ACP) over stdio rather than runtime-specific integrations.
- The community agy-buzz-acp bridge provides an ACP-over-stdio adapter for the
agy CLI, enabling Buzz to interact with Antigravity agents backed by Google AI
subscriptions.
