---
topic: compose-ui-testing
category: knowledge
tags: [knowledge, compose-ui-testing]
updated_at: 2026-11-24T07:02:38.980501+00:00
confidence: 0.95
---

# Knowledge: Compose-Ui-Testing

- Android Compose Preview Screenshot Testing records reference images via
`:app:updateDebugScreenshotTestDefaultTestSuite` and validates them via
`:app:testDebugScreenshotTestDefaultTestSuite`.
- In Compose screenshot previews using Coil, provide a
`LocalAsyncImagePreviewHandler` returning results with `DataSource.MEMORY_CACHE`
to bypass crossfade animations and avoid non-deterministic renders.
- In Robolectric Compose UI tests containing indeterminate animations such as
`CircularProgressIndicator`, pause the test clock
(`composeTestRule.mainClock.autoAdvance = false`) and flush frames explicitly
(`waitForIdle()`, `advanceTimeByFrame()`, `waitForIdle()`) so synchronization
does not hang.
- In Compose Robolectric tests, resolve Android string resources via
`ApplicationProvider.getApplicationContext<Application>()` rather than
`LocalContext` resource calls to satisfy Android Lint's
`LocalContextGetResourceValueCall` check.

- Annotate Robolectric pixel-inspection tests with
@GraphicsMode(GraphicsMode.Mode.NATIVE) so Canvas(bitmap) actually rasterizes
pixels; legacy ShadowCanvas returns 0 for Bitmap.getPixel().

- Use the Gradle task `:app:testDebugScreenshotTestDefaultTestSuite` to verify
Android Gradle Plugin Compose screenshot tests against reference PNGs.

- Compose Preview Screenshot Testing
(:app:updateDebugScreenshotTestDefaultTestSuite) renders GraphicsLayer and
chained RenderEffect blurs in reference images, while Robolectric tests
verifying custom draw/GraphicsLayer modifiers must explicitly pump draw frames.

- Robolectric Compose tests exercising D-pad traversal and focus ring rendering
require `InputMode.Keyboard` to trigger non-touch focus behavior and indicators.
- Compose 2D directional focus (`FocusDirection.Down`) resolves geometrically by
visual bounds rather than list index, so moving down from a horizontally offset
hero button into a horizontal card row focuses the vertically aligned card
rather than index 0.
