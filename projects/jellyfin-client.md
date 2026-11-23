---
topic: jellyfin-client
category: project
tags: [project, jellyfin-client]
updated_at: 2026-11-23T07:12:51.406573+00:00
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

- Fork of MakD/AFinity (upstream/master) using master as the default branch.
- Intentionally omits libmpv/MPV player support (MpvDeviceCheckItem,
MpvConfigEditorDialog) and the in-app self-updater (UpdateAvailableDialog);
during upstream merges, keep these removed and strip any newly added localized
strings.xml entries for MPV or self-update.
- Gradle verification suite consists of ktfmtCheck, checkCancellationRethrow,
checkItemProjection, compileDebugKotlin, and testDebugUnitTest, with
fork-specific composeLintChecks and mockk dependencies in
gradle/libs.versions.toml.
- Preserves fork-specific Compose recomposition optimizations
(contentAlphaProvider in HeroCarousel.kt, rememberCardRowListState() and
isScrollingProvider in HomeScreen.kt) and newer Gradle wrapper versions when
syncing with upstream.

- AppPinnedNavigationDrawer (PinnedNavigationDrawer.kt and PinnedDrawerBlur.kt)
uses borderless Material 3 tonal elevation (surfaceContainerHigh blended at
DefaultPinnedDrawerTintAlpha = 0.66f over a frosted blur backdrop) while
horizontal rows scroll edge-to-edge behind the drawer via
LocalPinnedDrawerStartInset.

- The full verification suite (`AGENTS.md`) runs `ktfmtCheck` (`ktfmtFormat` for
formatting), custom Gradle checks `checkCancellationRethrow` and
`checkItemProjection`, `compileDebugKotlin`, `testDebugUnitTest`,
`:app:lintDebug`, and `:app:testDebugScreenshotTestDefaultTestSuite`.
- JVM unit and Robolectric tests under `app/src/test/java/com/makd/afinity/` use
shared test utilities such as `testutil/FakeApiClient.kt` to test repositories,
Room database migrations, Audiobookshelf download workers
(`AbsMediaDownloadWorker`), and ViewModels, with JaCoCo configured to include
Robolectric class execution.

- Navigation defaults in NavigationPreferenceDefaults.kt (com.makd.afinity)
enable the navigation drawer by default (DEFAULT_NAVIGATION_DRAWER_ENABLED =
true) with individual library menu entries (DrawerLibrariesMode.EACH, preserving
legacy libraries_in_drawer = true without individual_libraries_in_drawer as
DrawerLibrariesMode.SCREEN).
- The side_sheet_enabled preference defaults to true and opens side sheets in
both landscape orientation and unfolded foldable postures.
- Gradle verification runs ktfmtCheckMain (which also checks untracked Kotlin
files under main source sets and is enforced by a git pre-push ktfmtCheck hook),
ktfmtCheckTest, custom static checks (checkCancellationRethrow,
checkItemProjection), testDebugUnitTest, lintDebug, and
testDebugScreenshotTestDefaultTestSuite.

- AppModalNavigationDrawer wraps Material 3 ModalWideNavigationRail and hosts
AppNavigationDrawerContent inside the header slot (with zeroed
expandedHeaderTopPadding, contentPadding, and windowInsets, plus
statusBarsPadding() on the inner header) so the drawer renders full-bleed
without a 20.dp trailing seam.
- Visual UI components and UX fixes should expose stateless composable overloads
decoupled from Hilt ViewModels and include @ScreenshotWindowPreviews coverage in
app/src/screenshotTest, updating reference PNGs via
:app:updateDebugScreenshotTestDefaultTestSuite and verifying with
:app:testDebugScreenshotTestDefaultTestSuite.

- In unfolded/expanded layouts, the pinned floating navigation drawer
(`PinnedNavigationDrawer.kt`, `shouldHidePinnedDrawerForRoute`) stays visible
across authenticated sub-screens (movie/TV/season/person details, search,
settings) and hides only on pre-auth/onboarding and full-screen
player/reader/viewer routes.
- Pinned drawer clearance (`272.dp` via `LocalPinnedDrawerStartInset` and
`LocalPinnedDrawerWindowInset`) is applied automatically by `AfinityScreen`
(`insetContentForPinnedDrawer = true`) and `pinnedDrawerComposable`; screens
with full-bleed horizontal carousels opt out
(`insetContentForPinnedDrawer = false`) and use `LocalDetailEdgePadding` or
`PortraitDetailLayout(edgeToEdgeItems = true)` (`StartInsetLazyListScope`) so
rows scroll edge-to-edge without double-padding non-bleed items.
- Movie, TV show, and season detail sub-screens use edge-to-edge horizontal
scrolling rows and a blurred top app bar consistent with the home screen.
- The full Gradle verification suite consists of `ktfmtCheck`,
`checkCancellationRethrow`, `checkItemProjection`, `compileDebugKotlin`,
`testDebugUnitTest`, `:app:lintDebug`, and
`:app:testDebugScreenshotTestDefaultTestSuite`.

- When navigation drawer mode is enabled on unfolded foldables and tablets
(`windowWidth >= 600.dp && windowHeight >=
500.dp` via `shouldPinNavigationDrawer`), `AppPinnedNavigationDrawer` pins a `240.dp`-wide floating card inset `16.dp` from start/top/bottom safe insets with `24.dp
` rounded corners and a semi-transparent backdrop blur (`PinnedDrawerBlur.kt`).
- `LocalPinnedDrawerStartInset` (`256.dp` when pinned, `0.dp` otherwise) is
added to horizontal row `contentPadding` (including `CardRow` and
`DiscoverSection`) so carousels start to the right of the drawer at rest while
scrolling behind the blurred card into the outer `16.dp` margins.
- Full verification for UI changes runs `ktfmtCheck`, `:app:lintDebug`,
Robolectric/unit tests, and `:app:testDebugScreenshotTestDefaultTestSuite`.

- Build artifacts (`base.archivesName`) and R8 mapping archives
(`archive<Variant>Mapping`) in `app/build.gradle.kts` use the
`jellyfin-client-v<versionName>` prefix instead of the legacy `afinity-` prefix.
- Because `app/build.gradle.kts` toggles `splits.abi` globally via
`isBundleBuild` whenever a `bundle*` task is present in the Gradle invocation,
`:app:bundleNightly` (and other `bundle*` tasks) must be run in a separate
`./gradlew` invocation from `:app:assembleDebug`, lint, and test tasks.

- Application ID is xyz.klinker.jellyfin (Kotlin base package com.makd.afinity);
pushing to origin master triggers Jenkins to build the nightly APK.
- Nightly builds enable <profileable android:enabled="true" android:shell="true"
/> via app/src/nightly/AndroidManifest.xml, wire
androidx.compose.runtime:runtime-tracing strictly under nightlyImplementation,
and gate custom PerfTrace helpers (PerfTrace.kt) on BuildConfig.IS_NIGHTLY
alongside ReportDrawnWhen/ReportDrawnAfter markers (StartupDrawReporting.kt).
- Compose compiler stability and skippability reports are generated with
-PcomposeMetrics=true alongside app/compose_compiler_config.conf.
- Standard Gradle verification runs ktfmtFormat/ktfmtCheck, custom tasks
checkCancellationRethrow and checkItemProjection, compileDebugKotlin,
testDebugUnitTest, :app:lintDebug, and
:app:testDebugScreenshotTestDefaultTestSuite.

- TopAppBar progressive blur (Modifier.topBarBlurBackdrop in
TopBarProgressiveBlur.kt) extends 16.dp (DefaultTopBarBlurBottomExtension) below
the bar height in draw/clip space without altering TopAppBar layout height or
touch targets; visual top-bar updates require regenerating reference screenshots
via :app:updateDebugScreenshotTestDefaultTestSuite.
- Project verification runs ktfmtCheck (:app:ktfmtCheckMain),
checkCancellationRethrow, checkItemProjection, lintDebug, testDebugUnitTest, and
:app:testDebugScreenshotTestDefaultTestSuite; because the local pre-push hook
runs ktfmtCheck across untracked workspace files, isolated commits from a dirty
workspace should be pushed from a temporary clean detached git worktree.

- Maintains a fork synced with upstream/master, enforcing Slack Compose lint
rules (such as ComposeModifierMissing and ComposeParameterOrder) via
app/lint-baseline.xml (refreshed with ./gradlew :app:updateLintBaselineDebug)
and storing screenshot test reference PNGs under
app/src/screenshotTestDefaultDebug/reference/.

- Home screen horizontal rows (CardRow, MusicVideoCardRow, MediaRowSkeleton)
must scroll edge-to-edge: keep outer section Columns unpadded horizontally,
apply Modifier.padding(horizontal = CardDimensions.RowGutter) to
HomeSectionHeader, and pass contentPadding = CardDimensions.RowContentPadding
(16.dp) to the LazyRow.
- Full Gradle verification suite runs ktfmtCheck, checkCancellationRethrow,
checkItemProjection, compileDebugKotlin, testDebugUnitTest, :app:lintDebug, and
:app:testDebugScreenshotTestDefaultTestSuite (requiring reference image updates
when UI layouts intentionally change).

- HomeScreen uses a scroll-driven progressive gradient backdrop blur (0.dp
bottom to 20.dp top) paired with a 0.25f vertical surface scrim behind
AfinityTopAppBar via Modifier.topBarBlurSource and Modifier.topBarBlurBackdrop
(TopBarProgressiveBlur.kt), replacing the previous opaque top bar cut.
- rememberTopBarScrollProgress (TopBarScrollProgress.kt) maps Home LazyListState
scroll position to a 0f..1f ramp over 450.dp when the hero carousel is present
(clamped to hero height on shorter viewports) or 64.dp when the hero is hidden
or offline.
