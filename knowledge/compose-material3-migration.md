---
topic: compose-material3-migration
category: knowledge
tags: [knowledge, compose-material3-migration]
updated_at: 2026-11-23T07:09:50.273856+00:00
confidence: 0.95
---

# Knowledge: Compose-Material3-Migration

- In Jetpack Compose Material 3 1.5.0 (`composeBom` 2026.11.01+), `Slider`
composables with custom `thumb` or `track` slots require migrating to
`Slider(state = SliderState(...), ...)`, and `SliderState.valueRange` references
migrate to `trackRange`.

- In Material 3 ModalWideNavigationRail, WideNavigationRailLayout reserves 20.dp
on the trailing edge of children in the content lambda
(WNRItemHorizontalPadding) and applies 44.dp default top padding
(WideNavigationRailDefaults.ContentPadding), which exposes a 20.dp strip of
modalContainerColor when rendering custom full-width drawer UI; place custom
full-bleed drawer content in the header slot instead with
expandedHeaderTopPadding = 0.dp, contentPadding = PaddingValues(0.dp), and
windowInsets = WindowInsets(0.dp) to span the full rail width.

- Migrating from Material 3 ListItem(headlineContent = { ... }) to the Material
3 Expressive ListItem(...) { ... } overload alters vertical row height metrics
(particularly noticeable at 150% font scale), requiring Compose screenshot test
reference updates.
