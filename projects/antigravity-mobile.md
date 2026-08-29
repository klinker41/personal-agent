---
topic: antigravity-mobile
category: project
tags: [project, antigravity-mobile]
updated_at: 2026-08-29T11:57:19.615906+00:00
confidence: 0.95
---

# Project: Antigravity-Mobile

- Architecture designed around MVVM and Jetpack Compose with unidirectional data
flow (UDF) using StateFlow and Kotlin Coroutines.
- Data layer uses a pluggable AgentRepository abstracting AgentDataSource
(RemoteAgentDataSource for REST/WebSocket streaming, ready for future
LocalAgentDataSource caching).
