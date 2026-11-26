---
topic: compose-ui-testing
category: knowledge
tags: [knowledge, compose-ui-testing]
updated_at: 2026-11-26T07:47:28.840738+00:00
compacted_at: 2026-11-26T07:47:28.840738+00:00
confidence: 0.95
---

# Compose UI Testing

## Preview Screenshot Testing

- Record AGP Compose preview reference PNGs with
  `:app:updateDebugScreenshotTestDefaultTestSuite` and verify against them
  with `:app:testDebugScreenshotTestDefaultTestSuite`.
- Screenshot previews render `GraphicsLayer` and chained `RenderEffect` blurs
  directly in reference images.
- For Coil images, provide a `LocalAsyncImagePreviewHandler` returning
  `DataSource.MEMORY_CACHE` results to bypass crossfade animations and ensure
  deterministic renders.

## Robolectric UI & Rendering

- **Animations & Custom Draw**: For indeterminate animations (e.g.,
  `CircularProgressIndicator`), set
  `composeTestRule.mainClock.autoAdvance = false` and flush frames manually
  (`waitForIdle()`, `advanceTimeByFrame()`, `waitForIdle()`) to avoid
  synchronization hangs. Explicitly pump draw frames when verifying custom draw
  or `GraphicsLayer` modifiers.
- **Pixel Inspection**: Annotate tests with
  `@GraphicsMode(GraphicsMode.Mode.NATIVE)` so `Canvas(bitmap)` rasterizes
  pixels (legacy `ShadowCanvas` returns `0` for `Bitmap.getPixel()`).
- **Lazy Containers**: Compact default viewports keep off-screen `LazyRow` and
  `LazyVerticalGrid` items uncomposed; scroll explicitly before asserting on
  or interacting with off-screen items.
- **String Resources**: Resolve Android strings via
  `ApplicationProvider.getApplicationContext<Application>()` instead of
  `LocalContext` resource calls to satisfy Android Lint's
  `LocalContextGetResourceValueCall` check.

## D-Pad & Focus Testing

- **Touch & Input Mode**: Configure `InstrumentationRegistry` touch-mode state
  in an outer JUnit rule (with `try`/`finally` cleanup for isolation) before
  `ComponentActivity` launches. Set `InputMode.Keyboard` (including under a
  touch-mode rule) to enable non-touch focus behavior, D-pad traversal, and
  focus rings.
- **Overlay Focus Restoration**: Activate trigger buttons via
  `Key.DirectionCenter` key events rather than touch `performClick()` so
  non-touch focus mode persists across overlay open/dismiss cycles.
- **2D Directional Geometry**: Directional focus (e.g., `FocusDirection.Down`)
  resolves geometrically by visual bounds rather than list index; moving down
  from a horizontally offset hero button into a horizontal card row focuses the
  vertically aligned card rather than index `0`.
