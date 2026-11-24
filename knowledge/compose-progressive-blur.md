---
topic: compose-progressive-blur
category: knowledge
tags: [knowledge, compose-progressive-blur]
updated_at: 2026-11-24T07:13:08.503974+00:00
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

- A multi-band full-resolution chained RenderEffect progressive blur (11 Skia
nodes) can be replaced with a single 0.25x downscaled blur pass (~16x fewer
pixels) masked via a BlendMode.DstIn smoothstep vertical alpha gradient (1.0
alpha over the top 55%, feathering to 0.0 over the crisp 1.0x contentLayer
underneath) to avoid 0dp-edge scaling artifacts while cutting GPU cost to 2
quarter-res nodes.
- Read scroll progress inside the draw phase when driving progressive top-bar
blurs to prevent scroll-time Compose recompositions.
