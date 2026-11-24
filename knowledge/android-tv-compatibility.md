---
topic: android-tv-compatibility
category: knowledge
tags: [knowledge, android-tv-compatibility]
updated_at: 2026-11-24T07:16:16.824406+00:00
confidence: 0.95
---

# Knowledge: Android-Tv-Compatibility

- Sharing a single `Activity` across mobile/tablet and Android TV requires
declaring optional `<uses-feature>` tags for `android.hardware.touchscreen` and
`android.software.leanback` (`android:required="false"`), setting
`android:banner`, and adding a `CATEGORY_LEANBACK_LAUNCHER` intent filter.
- Lowering Android `minSdk` from 35 to 34 causes JDK 21 `SequencedCollection`
methods like `List.removeFirst()` to fail Android Lint (`NewApi`, requires API
35); use `removeAt(0)` instead on API 34 targets.

- Requesting `android.permission.CHANGE_WIFI_MULTICAST_STATE` implicitly injects
a required `android.hardware.wifi` feature into the merged AndroidManifest;
explicitly declare `<uses-feature android:name="android.hardware.wifi"
android:required="false" />` and guard `WifiManager ` usage at runtime to keep
Ethernet-only devices compatible.
- Verify optional hardware features in merged manifests via
`aapt2 dump badging <apk>` (`uses-feature-not-required:
name='android.hardware.wifi'`) and unit-test them in Robolectric by inspecting
`PackageManager.getPackageInfo(...).reqFeatures` to ensure
`PackageManager.FEATURE_WIFI` lacks `FeatureInfo.FLAG_REQUIRED`.
