---
topic: compose-floating-overlays
category: knowledge
tags: [knowledge, compose-floating-overlays]
updated_at: 2026-11-23T07:09:22.826303+00:00
confidence: 0.95
---

# Knowledge: Compose-Floating-Overlays

- In Jetpack Compose, applying an empty `Modifier.pointerInput(Unit) {}` to a
floating overlay card blocks pointer hit-testing from falling through to
underlying `Box` siblings while leaving events unconsumed so child `clickable`
items still trigger clicks after sub-slop finger movement (unlike a manual
`awaitPointerEventScope` consumption loop).
- To let horizontal lists rest clear of a floating side overlay yet scroll
behind its blurred bounds and outer margins, expose the overlay's total start
occupancy via a `CompositionLocal` (annotated with
`@Suppress("ComposeCompositionLocalUsage")` to satisfy Compose lint) and add it
to each row's start `contentPadding`.

- Progressive blur backdrops that fade to zero at the bottom edge of a Compose
TopAppBar leave icons near the bottom with negligible blur; extend the draw/clip
backdrop height (e.g., +16.dp) independently of layout height so icons stay
visually backed without intercepting touch events below the bar.
