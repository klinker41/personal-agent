---
topic: android-hinge-sensor
category: knowledge
tags: [knowledge, android-hinge-sensor]
updated_at: 2026-10-25T06:01:19.394924+00:00
confidence: 0.95
---

# Knowledge: Android-Hinge-Sensor

- Android 11+ (API 30) exposes continuous foldable hinge angle telemetry (0° to
180° float stream) via Sensor.TYPE_HINGE_ANGLE on SensorManager, whereas Jetpack
WindowManager FoldingFeature only reports discrete states (FLAT, HALF_OPENED).
- On Samsung One UI foldable builds, continuous raw hinge angle sensor access is
restricted/throttled during screen transitions for standard apps, requiring
Shizuku (ADB privileges) to intercept telemetry from Samsung's built-in
interactive fold wallpaper service.
