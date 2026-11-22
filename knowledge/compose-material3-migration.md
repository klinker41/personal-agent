---
topic: compose-material3-migration
category: knowledge
tags: [knowledge, compose-material3-migration]
updated_at: 2026-11-22T07:20:35.556501+00:00
confidence: 0.95
---

# Knowledge: Compose-Material3-Migration

- In Jetpack Compose Material 3 1.5.0 (`composeBom` 2026.11.01+), `Slider`
composables with custom `thumb` or `track` slots require migrating to
`Slider(state = SliderState(...), ...)`, and `SliderState.valueRange` references
migrate to `trackRange`.
