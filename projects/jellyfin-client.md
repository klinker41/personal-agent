---
topic: jellyfin-client
category: project
tags: [project, jellyfin-client]
updated_at: 2026-11-22T07:23:50.672680+00:00
confidence: 0.95
---

# Project: Jellyfin-Client

- Kotlin Android Jellyfin client (klinker41/jellyfin-client) built with Gradle
Kotlin DSL (app/build.gradle.kts), Robolectric tests, and JaCoCo XML coverage
reporting (app/build/reports/coverage/test/debug/report.xml).
- Persistence layer uses Room with KSP-generated DAOs (UserDataDao, ServerDao,
CustomHomeSectionDao, MusicQueueDao) and SQLite schema migrations in
data/database/DatabaseMigrations.kt.

- Android Kotlin app (`com.makd.afinity`, `targetSdk = 37`) integrating Media3,
Google Cast, Audiobookshelf progress sync, and mDNS local service discovery
(`CHANGE_WIFI_MULTICAST_STATE` and API 37+ `ACCESS_LOCAL_NETWORK`).
- Full Gradle verification suite runs `./gradlew ktfmtCheck
checkCancellationRethrow checkItemProjection compileDebugKotlin
testDebugUnitTest :app:lintDebug :app:testDebugScreenshotTestDefaultTestSuite`.

- Android Jellyfin client in /workspace/jellyfin-client, maintained as a fork
(https://github.com/klinker41/jellyfin-client, origin
git@github.com:klinker41/jellyfin-client.git on branch master) of upstream
MakD/AFinity (https://github.com/MakD/AFinity).
- Enforces strict upstream mergeability (documented in AGENTS.md): keep edits
and indentation changes in upstream files minimal/surgical and isolate custom
logic into standalone files so future merges from MakD/AFinity stay
conflict-free.
- Implements sub-screen NavigationBar and NavigationRail slide animations via a
standalone NavigationSuiteState.kt helper wired into Material 3
NavigationSuiteScaffold(state = ...) in MainNavigation.kt rather than wrapping
or re-indenting upstream NavHost composable blocks.
- Build and verification pipeline runs Gradle ktfmtFormat, static checks, unit
tests, and assembleDebug.

- Android client customized from AFinity (source package `com.makd.afinity`)
using Gradle 9.8.2 and AGP 9 (`android.newDsl=true`), defining three `:app`
build types that output `arm64-v8a` and `armeabi-v7a` ABI-split APKs: `debug`
(`xyz.klinker.jellyfin.debug`), `release` (`xyz.klinker.jellyfin`, R8 minified),
and `nightly` (`xyz.klinker.jellyfin.nightly`, inherits `release` with launcher
label 'Jellyfin Nightly' and `IS_NIGHTLY = true`).
- Defines custom static verification tasks in `app/build.gradle.kts` alongside
`ktfmtCheck` (`kotlinLangStyle()`): `checkCancellationRethrow` (ensures
`catch (e: Exception)` blocks rethrow `CancellationException`) and
`checkItemProjection` (ensures screens do not hold raw item copies in mutable
`UiState` without `ItemStore.project`).
- Jenkins CI job (`jellyfin-client` on `unraid-nas` at
`/mnt/user/docker/jenkins/home/jobs/jellyfin-client/config.xml`, executed on
`jenkins-android-agent`) runs `./gradlew --no-daemon ktfmtCheck
checkCancellationRethrow checkItemProjection lintDebug testDebugUnitTest
app:testDebugScreenshotTestDefaultTestSuite` and `./gradlew --no-daemon
assembleDebug assembleRelease
assembleNightly`, archiving only the `arm64-v8a` APKs for `release` and `nightly
`.

- Fork of `MakD/AFinity` (configured as `upstream` on default branch `master`,
Antigravity project ID `1b308a30-734d-4f96-9d7c-ebcc3699d0da`), verified via
`./gradlew ktfmtCheck checkCancellationRethrow checkItemProjection
compileDebugKotlin testDebugUnitTest`.
- Upstream commits from `MakD/AFinity` are automatically merged, validated, and
pushed every 12 hours by the `jellyfin-upstream-sync` sidecar in
`antigravity-plugin`, with Slack alerts sent only on failures or dirty/unpushed
local state.

- All UI changes must work across all screen sizes—including phones, foldables
(folded and unfolded), and tablets—across compact, medium, and expanded window
size classes without hard-coded phone-portrait assumptions (documented in
AGENTS.md).

- Repository klinker41/jellyfin-client is an Android Kotlin project that
enforces upstream mergeability guidelines (per AGENTS.md), avoiding broad
refactors to app/src/main/ and keeping test additions isolated under
app/src/test/.
- Core playback and casting classes include ui/player/PlayerViewModel.kt,
data/repository/playback/JellyfinPlaybackRepository.kt, player/mpv/MPVPlayer.kt,
player/AudioService.kt, and cast/CastManager.kt.
- Debug JaCoCo unit test coverage reports are generated with the
:app:createDebugUnitTestCoverageReport Gradle task.

- Fork of upstream `MakD/AFinity` (`upstream/master`); avoid repo-wide
formatting or lint cleanups on existing upstream `.kt` and `.xml` files to
minimize upstream merge conflicts.
- Kotlin formatting is enforced via `com.ncorti.ktfmt.gradle`
(`kotlinLangStyle()`) using `./gradlew ktfmtCheck` and gated locally by
`.githooks/pre-push` (`setup-git.sh` sets
`git config core.hooksPath .githooks`).
- Static analysis uses Android Lint plus
`com.slack.lint.compose:compose-lint-checks` in `app/build.gradle.kts` with
`baseline = file("lint-baseline.xml")`, `warningsAsErrors = true`, and
`abortOnError = true`, while disabling upstream/release churn checks
(`GradleDependency`, `NewerVersionAvailable`, `AndroidGradlePluginVersion`,
`MissingTranslation`, `ExtraTranslation`, `ImpliedQuantity`). Refresh the
baseline after upstream merges with `./gradlew :app:updateLintBaselineDebug`.
- CI verification runs `ktfmtCheck`, `checkCancellationRethrow`,
`checkItemProjection`, `compileDebugKotlin`, `testDebugUnitTest`, and
`:app:lintDebug`.

- Source repository is https://github.com/klinker41/jellyfin-client.git (master
branch), targeting Android SDK android-37.2 with build-tools/37.0.0 and JDK 21.
- Jenkins CI job (https://jenkins.klinker.computer/job/jellyfin-client/, config
at /mnt/user/docker/jenkins/home/jobs/jellyfin-client/config.xml on unraid-nas)
polls master every 15 minutes (H/15 * * * *) on unraid-server-android-agent,
runs testDebugUnitTest, testReleaseUnitTest, and assembleRelease, archives
release APKs (app/build/outputs/apk/release/*.apk) for the past 50 builds,
publishes JUnit test results, and sends Slack notifications on every failure.

- Android Jellyfin client fork (`klinker41/jellyfin-client`, package
`com.makd.afinity`, default branch `master`) built with Jetpack Compose, Room
(KSP), Dagger/Hilt, Coil, and DataStore, with GitHub Issues enabled on the fork
and Play Store alpha publishing replacing the upstream GitHub self-updater.
- Build verification enforces `ktfmtCheck` (also run via git pre-push hook;
formatted with `ktfmtFormat`), custom static Gradle tasks
(`checkCancellationRethrow`, `checkItemProjection`), `:app:lintDebug`,
unit/Robolectric tests (`testDebugUnitTest` using `kotlinx-coroutines-test`,
`mockk`, and `turbine`), and JaCoCo coverage
(`:app:createDebugUnitTestCoverageReport`).
- Compose Preview Screenshot Testing
(`:app:testDebugScreenshotTestDefaultTestSuite` /
`:app:updateDebugScreenshotTestDefaultTestSuite`) uses `ScreenshotPreviewHost`
and `ScreenshotPreviewAnnotations` to validate component and full-screen
responsive previews across phone (portrait/dark, landscape, 150% font scale),
unfolded foldable, and tablet (portrait/landscape) configurations.

- Jenkins CI builds assembleDebug, assembleRelease, and assembleNightly on
unraid-server-android-agent; because gradle.properties sets
org.gradle.parallel=true and org.gradle.jvmargs=-Xmx4096m, running both
R8-minified Release and Nightly variants in a single Gradle invocation can
exhaust JVM heap in :app:minifyNightlyWithR8 unless split into separate
invocations or given a larger heap.

- Fork of AFinity configured with applicationId `xyz.klinker.jellyfin` (with
`.debug` and `.nightly` build-type suffixes) while retaining the upstream Kotlin
namespace (`com.makd.afinity`) to preserve upstream mergeability; macrobenchmark
targets (`BaselineProfileGenerator`, `StartupBenchmark`) use
`xyz.klinker.jellyfin`.
- Isolates fork-specific string resources such as `@string/launcher_name`
(`Jellyfin` with `translatable="false"`, overridden via `resValue` to
`Jellyfin Nightly` in nightly builds) in
`app/src/main/res/values/fork_strings.xml` so launcher branding applies across
all locales without conflicting with upstream `strings.xml` or Weblate
translations.

- Offline media download and local persistence in `klinker41/jellyfin-client`
are handled by WorkManager workers in `data/workers/` (`MediaDownloadWorker`,
`AbsMediaDownloadWorker`),
`data/repository/download/JellyfinDownloadRepository.kt`, and Room DAOs wrapped
by `data/repository/impl/DatabaseRepositoryImpl.kt`.

- Manages dependencies in `gradle/libs.versions.toml` alongside the Gradle
wrapper and Android SDK packages, gated by `ktfmtCheck`, static checks, unit
tests, Android Lint, and screenshot tests.

- Publishes to Google Play under application ID `xyz.klinker.jellyfin`; the
in-app GitHub self-updater and `REQUEST_INSTALL_PACKAGES` permission were
removed for Play Store compliance while keeping the read-only release notes
viewer and cancelling legacy `update_check_worker` WorkManager jobs in
`AfinityApplication.kt`.
- Nightly Google Play `alpha` track publishing is automated via
`fastlane android publish_nightly` (setup documented in
`docs/PLAY_STORE_PUBLISHING.md`) and executed by the `jellyfin-client` Jenkins
job on `unraid-nas` when `PLAY_SERVICE_ACCOUNT_JSON` and `ANDROID_KEYSTORE_*`
environment variables are present, falling back to
`./gradlew --no-daemon bundleNightly` otherwise.

- Repository (klinker41/jellyfin-client) data layer centers on
JellyfinMediaRepository, AppDataRepository, and HomeSectionsRepository backed by
ItemStore caching and the Jellyfin SDK ApiClient, alongside
JellyfinMusicRepository, AudiobookshelfRepositoryImpl, JellyseerrRepositoryImpl,
JellyfinAdminRepository, and JellyfinServerAdminRepository.
- Repository unit tests should target a fake ApiClient or MockWebServer rather
than mockkConstructor coupling to Jellyfin SDK API classes, with coverage
measured via JaCoCo.

- Playback uses ExoPlayer exclusively after removing `libmpv`, its native
libraries, preferences, and player-engine selection logic (cutting the
`arm64-v8a` APK from 75.4 MiB to 33.0 MiB and estimated Play Store download from
~33.5 MiB to ~14.9 MiB).
- Typography loads `AppFont` families (`Google Sans Flex`, `Quicksand`,
`IBM Plex Sans`, `IBM Plex Sans Condensed`) on demand via GMSCore Downloadable
Fonts (`androidx.compose.ui:ui-text-google-fonts`), keeping only
`inter_bold_digits.ttf` bundled for ranked cards while `RemoteViews` audio
widgets use system `sans-serif`.
- Core library desugaring must remain enabled despite `minSdk = 35` to satisfy
Readium's AAR metadata requirement, and `slf4j-api` must stay on the runtime
classpath for startup initialization prior to `SdkLogBridge.install()`.
