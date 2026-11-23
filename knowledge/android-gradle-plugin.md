---
topic: android-gradle-plugin
category: knowledge
tags: [knowledge, android-gradle-plugin]
updated_at: 2026-11-23T07:09:50.273375+00:00
confidence: 0.95
---

# Knowledge: Android-Gradle-Plugin

- In Android Gradle Plugin (AGP) 9 with `android.newDsl=true`, unit test tasks
are only generated for the designated tested build type (`testDebugUnitTest` by
default); `testReleaseUnitTest` is not created and invoking it fails Gradle task
selection.

- To enforce strict Android and Jetpack Compose
(`com.slack.lint.compose:compose-lint-checks`) lint rules on new code in a fork
without touching upstream files, configure AGP's `lint {}` block with
`baseline = file("lint-baseline.xml")`, `warningsAsErrors = true`, and
`abortOnError = true`, and generate or refresh the baseline via
`./gradlew :app:updateLintBaselineDebug`.
- When using `warningsAsErrors = true` in Android Lint, disable external-churn
checks (`GradleDependency`, `NewerVersionAvailable`,
`AndroidGradlePluginVersion`, `MissingTranslation`, `ExtraTranslation`,
`ImpliedQuantity`) so newly published library versions or translation syncs do
not break `:app:lintDebug` or `:app:lintVitalRelease` builds.

- Running multiple R8-minified Android build variants (such as assembleRelease
and assembleNightly) in a single Gradle invocation with org.gradle.parallel=true
runs concurrent minify*WithR8 tasks (including KeepRadiusHtmlReportGenerator)
inside the same JVM heap, causing Java heap space OOMs at -Xmx4096m; split
minified variants into separate ./gradlew invocations or increase
-Dorg.gradle.jvmargs (e.g. -Xmx8192m).

- In AGP 9.5.1+, enabling `splits.abi` conflicts with Android App Bundle
(`bundle*`) tasks and must be conditionally disabled when building AABs.
- In module-level `app/build.gradle.kts`, resolve keystore file paths from
environment variables (such as `ANDROID_KEYSTORE_FILE`) using
`rootProject.file(...)` instead of `file(...)` so paths relative to the
repository root resolve accurately.

- At `minSdk = 35`, AGP packages `.so`, `.dex`, and `resources.arsc`
uncompressed (`STORED`) in APKs for direct `mmap`, making on-disk APK sizes
roughly 2x larger than actual Play Store downloads; estimate Play delivery size
by summing compressed base sections in the `.aab` while excluding
`BUNDLE-METADATA/` (such as `proguard.map`).
- Running `bundle<Variant>` and `assemble<Variant>` in the same Gradle
invocation causes AGP to build a universal APK instead of per-ABI split APKs;
invoke `assemble<Variant>` separately when generating or measuring per-ABI APKs.

- To include Robolectric test execution in JaCoCo unit test coverage reports,
configure `JacocoTaskExtension` (inside `plugins.withId("jacoco")`) with
`isIncludeNoLocationClasses = true` and `excludes = listOf("jdk.internal.*")`.

- When an Android build script conditionally toggles `splits.abi` based on
whether `bundle*` tasks are in the current Gradle invocation, combining
`bundle*` and `assemble*` (or debug resource/test tasks) in a single `./gradlew`
run causes `packageDebug` `NullPointerException` failures; split
APK/verification tasks and AAB bundle tasks into separate `./gradlew`
invocations.

- Run ./gradlew :app:updateLintBaselineDebug (updateLintBaseline<Variant>) to
regenerate lint-baseline.xml in place, baselining newly introduced lint issues
while automatically pruning resolved or stale entries.
