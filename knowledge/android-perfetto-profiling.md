---
topic: android-perfetto-profiling
category: knowledge
tags: [knowledge, android-perfetto-profiling]
updated_at: 2026-11-23T07:07:50.028239+00:00
confidence: 0.95
---

# Knowledge: Android-Perfetto-Profiling

- Keep production release APKs free of tracing overhead by declaring
<profileable android:enabled="true" android:shell="true" /> in a dedicated
build-type manifest overlay (e.g., src/nightly/AndroidManifest.xml), scoping
androidx.compose.runtime:runtime-tracing to that variant's implementation
configuration, and gating custom Trace wrappers on BuildConfig.
- When capturing Perfetto traces directly from https://ui.perfetto.dev over
Android WebUSB, run `adb kill-server` on the host machine first if Chrome
reports the USB device is busy or claimed.
- In the Perfetto Web UI, surfacing jank and custom app trace sections requires
enabling CPU Scheduling details, Android Frame timeline, and Atrace userspace
annotations (gfx, input, view, wm, am, sm, hal, res, dalvik) with the target
applicationId explicitly added under 'Record events from specific apps'—without
the package filter, custom Trace slices and ReportDrawnWhen/ReportDrawnAfter
events will not appear.
