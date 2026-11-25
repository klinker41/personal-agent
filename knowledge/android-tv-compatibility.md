---
topic: android-tv-compatibility
category: knowledge
tags: [knowledge, android-tv-compatibility]
updated_at: 2026-11-25T07:15:35.147414+00:00
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

- In a universal phone/tablet/TV APK, do not attach `CATEGORY_HOME` and
`CATEGORY_DEFAULT` directly to `MainActivity` or phones and tablets will prompt
to use the app as their home screen; instead declare a disabled-by-default
`<activity-alias>` (`android:enabled="false"`) targeting `MainActivity` and
toggle it via `PackageManager.setComponentEnabledSetting` only on TV devices.
- Android TV home launchers require `launchMode="singleTask"` (typically with
`stateNotNeeded="true"`) and `onNewIntent` handling for `CATEGORY_HOME` to pop
the `NavController` back to the home destination rather than stacking duplicate
activity instances on Home button presses.
- Before disabling a custom `CATEGORY_HOME` `<activity-alias>`, check
`PackageManager` for at least one other enabled `CATEGORY_HOME` launcher; if the
stock launcher was disabled via `adb shell pm disable-user --user 0 <pkg>`
(instead of switching defaults via `cmd package set-home-activity`), disabling
the alias would leave the TV with zero launchers and should be blocked until the
stock launcher package is re-enabled (`adb shell pm enable <pkg>`).
- On Android 11+ (API 30+), an Android TV launcher must declare `<queries>`
intents for `CATEGORY_LEANBACK_LAUNCHER`, `CATEGORY_LAUNCHER`, and
`CATEGORY_HOME` to discover installed apps and active launchers, plus explicit
`<package>` entries for stock launcher package names (e.g.,
`com.google.android.apps.tv.launcherx`) so disabled stock launchers remain
visible when checking recovery options.
