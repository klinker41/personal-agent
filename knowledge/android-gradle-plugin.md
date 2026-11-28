---
topic: android-gradle-plugin
category: knowledge
tags: [knowledge, android-gradle-plugin]
updated_at: 2026-11-28T07:51:18.634857+00:00
compacted_at: 2026-11-28T07:51:18.634857+00:00
confidence: 0.95
---

# Knowledge: Android-Gradle-Plugin

## Build & Signing

- **Keystore paths**: In module-level `app/build.gradle.kts`, resolve keystore
  paths from environment variables (e.g., `ANDROID_KEYSTORE_FILE`) with
  `rootProject.file(...)` instead of `file(...)` so repository-root-relative
  paths resolve accurately.
- **Parallel R8 OOMs**: Running multiple R8-minified variants (e.g.,
  `assembleRelease` and `assembleNightly`) in one `./gradlew` invocation with
  `org.gradle.parallel=true` executes concurrent `minify*WithR8` tasks
  (including `KeepRadiusHtmlReportGenerator`) in the same JVM heap, causing
  Java heap space OOMs at `-Xmx4096m`. Split minified variants into separate
  `./gradlew` invocations or raise `-Dorg.gradle.jvmargs` (e.g., `-Xmx8192m`).

## ABI Splits, App Bundles & APK Size

- **Separate AAB and APK/verification runs**: In AGP 9.5.1+, `splits.abi`
  conflicts with Android App Bundle (`bundle*`) tasks and must be
  conditionally disabled when building AABs. Always run `bundle*` tasks in a
  separate `./gradlew` invocation from `assemble*` and debug resource/test
  tasks: combining them makes AGP emit a universal APK instead of per-ABI
  split APKs (breaking per-ABI generation or measurement), and when
  `splits.abi` is conditionally toggled on `bundle*` tasks, triggers
  `packageDebug` `NullPointerException` failures.
- **Play delivery size at `minSdk = 35`**: AGP packages `.so`, `.dex`, and
  `resources.arsc` uncompressed (`STORED`) in APKs for direct `mmap`, making
  on-disk APKs roughly 2x larger than Play Store downloads. Estimate Play
  delivery size by summing compressed base sections in the `.aab` while
  excluding `BUNDLE-METADATA/` (such as `proguard.map`).

## Lint

- **Strict fork lint with baseline**: To enforce strict Android and Jetpack
  Compose (`com.slack.lint.compose:compose-lint-checks`) rules on new code in
  a fork without modifying upstream files, configure AGP's `lint {}` block
  with `baseline = file("lint-baseline.xml")`, `warningsAsErrors = true`, and
  `abortOnError = true`. Run `./gradlew :app:updateLintBaselineDebug`
  (`updateLintBaseline<Variant>`) to generate or regenerate
  `lint-baseline.xml` in place, baselining new issues while automatically
  pruning resolved or stale entries.
- **Disable external-churn checks**: When `warningsAsErrors = true`, disable
  `GradleDependency`, `NewerVersionAvailable`, `AndroidGradlePluginVersion`,
  `MissingTranslation`, `ExtraTranslation`, and `ImpliedQuantity` so newly
  published library versions or translation syncs do not break
  `:app:lintDebug` or `:app:lintVitalRelease`.

## Unit Tests & Coverage

- **AGP 9 tested build type**: With `android.newDsl=true`, AGP 9 generates
  unit test tasks only for the designated tested build type
  (`testDebugUnitTest` by default); `testReleaseUnitTest` is not created and
  invoking it fails Gradle task selection.
- **Corrupted test-result store**: Interrupted or canceled unit test tasks
  (e.g., `testDebugUnitTest`) can leave corrupted binary files under
  `build/test-results/<variant>/binary` that crash subsequent runs with
  `java.io.EOFException`; run `cleanTest<Variant>UnitTest` to reset the store.
- **Robolectric JaCoCo coverage**: To include Robolectric execution in JaCoCo
  unit test coverage reports, configure `JacocoTaskExtension` inside
  `plugins.withId("jacoco")` with `isIncludeNoLocationClasses = true` and
  `excludes = listOf("jdk.internal.*")`.
