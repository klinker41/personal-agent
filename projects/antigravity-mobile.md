---
topic: antigravity-mobile
category: project
tags: [project, antigravity-mobile]
updated_at: 2026-10-05T00:01:04.485970+00:00
confidence: 0.95
---

# Project: Antigravity-Mobile

- Architecture designed around MVVM and Jetpack Compose with unidirectional data
flow (UDF) using StateFlow and Kotlin Coroutines.
- Data layer uses a pluggable AgentRepository abstracting AgentDataSource
(RemoteAgentDataSource for REST/WebSocket streaming, ready for future
LocalAgentDataSource caching).

- Architecture connects to the container hub via `/gateway/v1` REST endpoints
and `/gateway/v1/stream` multiplexed WebSocket rather than calling the internal
hub directly, omitting push notifications for MVP.
