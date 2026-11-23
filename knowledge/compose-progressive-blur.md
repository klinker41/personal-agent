---
topic: compose-progressive-blur
category: knowledge
tags: [knowledge, compose-progressive-blur]
updated_at: 2026-11-23T07:12:51.409947+00:00
confidence: 0.95
---

# Knowledge: Compose-Progressive-Blur

- To implement a performant progressive gradient backdrop blur in Jetpack
Compose, bypass GraphicsLayer recording and FBO allocation entirely when
progress == 0f, record the scrolling content once into a shared GraphicsLayer
when active, and render only a cropped top strip (barHeight + blur padding) in
the backdrop layer with a multi-band chained hardware RenderEffect pyramid.
- When driving a cropped backdrop GraphicsLayer from a LazyListState, track a
scroll position key separately from the clamped 0f..1f progress ramp so the
backdrop continues refreshing after progress reaches 1f, and invalidate the
backdrop on modifier detach, reposition, or state swap.
