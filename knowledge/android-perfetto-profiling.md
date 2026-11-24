---
topic: android-perfetto-profiling
category: knowledge
tags: [knowledge, android-perfetto-profiling]
updated_at: 2026-11-24T07:13:08.503557+00:00
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

- When starting a background trace via 'adb shell ... perfetto --txt -c - -o
<path> --background', perfetto prints its shell-owned PID; running 'killall -INT
perfetto' fails with 'Operation not permitted' on system-owned perfetto
processes and leaves the trace file at 0 bytes—signal the returned PID directly
instead.
- Some on-device perfetto binaries reject 'fill_behavior' in textproto (--txt)
configs as an unknown option; omit it for portable pbtx trace configs.
- On high-resolution 120Hz foldables, high frame buffer stuffing paired with low
Main Thread Choreographer#doFrame duration (~1.3ms) points to a RenderThread/GPU
bottleneck—check RenderThread Drawing, flush layers, Skia OpsTask per frame,
offscreen DstIn saveLayers, and ShadowCircularRRectOp counts in trace_processor.
