---
topic: compose-ui-testing
category: knowledge
tags: [knowledge, compose-ui-testing]
updated_at: 2026-11-22T07:15:27.439069+00:00
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
