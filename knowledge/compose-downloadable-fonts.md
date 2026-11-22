---
topic: compose-downloadable-fonts
category: knowledge
tags: [knowledge, compose-downloadable-fonts]
updated_at: 2026-11-22T07:23:50.673688+00:00
confidence: 0.95
---

# Knowledge: Compose-Downloadable-Fonts

- `androidx.compose.ui:ui-text-google-fonts` (included in the Compose BOM) loads
shared fonts from GMSCore (`com.google.android.gms.fonts`) using only an XML
provider certificate array, eliminating bundled TTF files and falling back to
system fonts when offline on first run or on non-GMS devices.
- App widgets built with `RemoteViews` cannot use Compose `ui-text-google-fonts`
(and XML downloadable fonts in third-party launchers can be unreliable), so
widgets should either use system `sans-serif` or retain a bundled font.
