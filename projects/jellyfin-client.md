---
topic: jellyfin-client
category: project
tags: [project, jellyfin-client]
updated_at: 2026-11-25T07:15:35.143924+00:00
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

- Audio and podcast playback is coordinated across AudioMiniPlayerOverlay,
MusicPlaybackManager, AudioService, and PodcastProgressTracker;
resolvePodcastProgress treats positions below 10,000 ms as 0 ticks (clearing
episode resume progress), so stop(), pauseForCast(), and track transitions must
capture and report the outgoing playback position before clearMediaItems()
resets state to 0L.

- In TV mode (`LocalIsTelevision.current`), `shouldPinNavigationDrawer` in
`PinnedNavigationDrawer.kt` returns `false` so the navigation drawer starts
collapsed (accessible via the top-left menu button and
`AppModalNavigationDrawer`) while keeping `HeroCarousel` enabled without
modifying upstream `MainNavigation.kt`.
- TV mode defaults initial screen focus to the `HeroCarousel` Play button across
all four hero layouts (`HeroContentRich`, `HeroContentCentered`,
`HeroPeekContentRich`, and `HeroPeekContentCentered`) via
`rememberHeroPlayFocusRequester` guarded by `rememberSaveable`, and uses
`focusOutline` across media and detail cards for D-pad focus highlights.
- Full Gradle verification runs `ktfmtCheck`, `checkCancellationRethrow`,
`checkItemProjection`, `compileDebugKotlin`, `testDebugUnitTest`,
`:app:lintDebug`, and `:app:testDebugScreenshotTestDefaultTestSuite`.

- Podcast album pages reuse MusicAlbumScreen and MusicAlbumViewModel, using
PodcastProgressTracker.isPodcastAlbum(albumId) (with offline/legacy cache
fallbacks) to expose MusicAlbumUiState.isPodcast and route episode clicks to
PodcastEpisodeDetailScreen via Destination.createPodcastEpisodeDetailRoute
instead of starting immediate playback.
- Full Gradle verification suite runs ktfmtCheck (ktfmtFormat to format), custom
checks checkCancellationRethrow and checkItemProjection, compileDebugKotlin,
testDebugUnitTest, :app:lintDebug, and
:app:testDebugScreenshotTestDefaultTestSuite
(:app:updateDebugScreenshotTestDefaultTestSuite to update reference
screenshots).

- Offline auto-download (com.makd.afinity.data.autodownload) combines a 4-hour
periodic WorkManager sync (AutoDownloadScheduler / AutoDownloadWorker),
15-minute throttled foreground syncs in AutoDownloadManager (cold launch, server
reconnect, exiting Offline Mode, and session or library preference changes), and
immediate syncs on Jellyfin WebSocket mediaChangeManager.itemsAdded events
(debounced by 2s and bypassing the 15-minute throttle via
scheduler.triggerSyncNow(wifiOnly, replace = true) when online, authenticated,
permitted by canDownload, and auto-download libraries are enabled).
- Gradle verification suite includes ktfmtFormat/ktfmtCheck, custom static
checks checkCancellationRethrow and checkItemProjection, compileDebugKotlin,
testDebugUnitTest, :app:lintDebug, and
:app:testDebugScreenshotTestDefaultTestSuite.

- Product goal is to eventually replace Symphonium as a unified Android client
for both video and audio content (music, audiobooks, and podcasts).
- Continue Listening (JellyfinMediaRepository, PodcastHomeHelpers,
PodcastHomeSections) merges and deduplicates in-progress AudioBook and podcast
Audio items sorted by lastPlayedDate, querying Items with Filters=IsResumable
instead of /Items/Resume so libraries hidden by LatestItemsExcludes are still
included.
- Latest Podcasts is a configurable HomeRow backed by AppDataRepository that
caches and displays unplayed episodes from podcast libraries, preferring
episode-embedded artwork over podcast-wide show artwork when available.
- Podcast playback progress is tracked and synced via PodcastProgressTracker and
MusicProgressReporter.
- Full Gradle verification pipeline includes ktfmtCheck,
checkCancellationRethrow, checkItemProjection, compileDebugKotlin,
testDebugUnitTest, :app:lintDebug, and
:app:testDebugScreenshotTestDefaultTestSuite.

- Supports Android TV and D-pad/keyboard navigation directly on the shared
`MainActivity` and `PlayerActivity` architecture (`LEANBACK_LAUNCHER`,
`Context.isTelevision()` / `LocalIsTelevision`, pure-Compose focus
rings/outlines, and `PlayerRemoteKeyHandler`) rather than a separate TV
activity, preserving compatibility with future Kotlin Multiplatform and web
keyboard navigation goals.
- Uses `minSdk = 34` (lowered from 35 for Android TV device compatibility) with
API 35 features such as `PdfRenderer` text extraction in `BookPages.kt` guarded
by SDK version checks.

- Publishes signed release bundles (`:app:bundleRelease`) for application ID
`xyz.klinker.jellyfin` to the Google Play `alpha` track via
`fastlane android publish_release`, using `PLAY_RELEASE_STATUS` (`completed` by
default, or `draft` while in Play Console Draft state).
- Jenkins job (`/mnt/user/docker/jenkins/home/jobs/jellyfin-client/config.xml`
on `unraid-nas`) binds `PLAY_SERVICE_ACCOUNT_JSON`
(`google-play-service-account` in GCP project `562752253464`) and
`ANDROID_KEYSTORE_*` credentials to run `fastlane android publish_release`,
falling back to `./gradlew bundleRelease`.

- LibraryPlaybackSpeedStore persists playback speeds (0.5x–3.0x, defaulting to
1.0x) in DataStore per library using keys playback_speed_abs_<libraryId>
(falling back to mediaType) for Audiobookshelf and playback_speed_jf_<libraryId>
for Jellyfin audiobook and podcast libraries, while caching item-to-library
mappings for offline playback.
- Jellyfin podcast playback speed resolution uses
PodcastProgressTracker.resolvePodcastTrack() and persisted albumId-to-libraryId
mappings in AutoDownloadPreferences so multiple podcast libraries maintain
independent speeds online and offline without affecting regular music playback.
- AudioService.playResolved() re-applies persisted library playback speed after
ensurePlayerInitialized() so speeds survive player tearDown(), and
MusicCastRemote gates the Cast lyrics overlay with !playbackState.isPodcast.

- Auto-download architecture (com.makd.afinity.data.autodownload and
AutoDownloadWorker): AutoDownloadManager, AutoDownloadPolicy,
AutoDownloadPreferences, AutoDownloadScheduler, and PlayedItemEventBus manage
per-(serverId, userId) library auto-downloads for unplayed and in-progress items
with a 6-hour retry cooldown on failed downloads.
- Auto-download scoping and playback cleanup: Tracks per-library auto-managed
item IDs separately so manual downloads remain untouched, caches podcast album
IDs for offline podcast tracking (mapping Music libraries for podcasts/music and
Books libraries for audiobooks), and automatically deletes auto-managed
downloads upon playback completion via PlayedItemEventBus hooks in
PodcastProgressTracker and JellyfinAudiobookReporter.
- Gradle verification suite runs ktfmtCheck (formatted via ktfmtFormat), custom
checks checkCancellationRethrow and checkItemProjection, compileDebugKotlin,
testDebugUnitTest, :app:lintDebug, and
:app:testDebugScreenshotTestDefaultTestSuite.

- Home screen Latest Audiobooks and Latest Podcasts shelves sort items
newest-first via sortedByRecency() in PodcastHomeHelpers.kt (using premiereDate
and dateCreated on AfinityTrack and AfinityAudiobook) inside AppDataRepository
(applyBookShelves, applyPodcastShelves, and refreshPlaybackSections).
- Selecting a podcast episode on the home screen navigates to
PodcastEpisodeDetailScreen (PodcastEpisodeDetailViewModel) instead of starting
playback immediately, using ItemStore.project with offline DB fallback and
exposing watched, favorite, and download controls.

- Podcast episodes (stored as Jellyfin Audio items) bypass SessionApi playback
start/progress/stop calls in MusicProgressReporter and CastManager; instead,
PodcastProgressTracker syncs resume state via UserDataApi using
syncPeriodicIfDue and an allowCompletion flag so periodic syncs and pauses near
the end of an episode do not mark it played prematurely.
- WebSocket UserDataChanged events directly drive Continue Listening and Latest
Podcasts visibility in HomeViewModel as well as played-item download deletion
(isMarkedPlayedEvent) in AutoDownloadManager, so podcast and audiobook progress
reporting in PlaybackRepository and JellyfinPlaybackRepository must avoid
emitting transient Played = true or zeroed PlaybackPositionTicks states during
active playback and pause.

- Uses a flat, zero-shadow UI design across dark and light themes: card
elevation shadows and the AppPinnedNavigationDrawer drop shadow are removed
outright without replacement borders/outlines for visual consistency and to
eliminate ShadowCircularRRectOp GPU overhead.
- PinnedDrawerBlur renders on a 0.25x downscaled layer, and
TopBarProgressiveBlur uses a single-pass 0.25x 20dp blur masked with a
smoothstep vertical alpha gradient (holding 1.0 alpha across the top 55% and
feathering to 0.0 at 100%) over full-res content, locking unfolded 2076x2152
scrolling at 120Hz with zero jank.
- HeroCarousel and PortraitDetailLayout avoid offscreen alpha = 0.99f +
BlendMode.DstIn saveLayers, HeroTone decodes downsampled 160x90 bitmaps, and
UserImageStore downsamples and caches user avatars as 256px JPEGs using unique
temp files.
- SessionManager uses optimistic startup session restoration (skipping initial
refreshUserPolicy) with asynchronous server address validation on sessionScope,
refreshing user policy once after validation and persisting updated credentials
when a server URL switches.
- CastManager offloads Dagger instantiation and Google Play Services
availability checks onto Dispatchers.IO before switching to
Dispatchers.Main.immediate for CastContext.getSharedInstance, which enforces
main-thread invocation.
- The repository's git pre-push hook runs ktfmtCheckMain across the workspace
and blocks pushes if unrelated untracked WIP Kotlin files are unformatted.

- PlayerViewModel handles container and decode PlaybackExceptions (such as
ERROR_CODE_PARSING_CONTAINER_UNSUPPORTED) via retryWithTranscodeFallback(),
setting forceTranscodeFallback = true and reloading media with enableDirectPlay
= false.
- Full Gradle verification suite runs ktfmtCheck, static checks,
compileDebugKotlin, testDebugUnitTest, :app:lintDebug, and
:app:testDebugScreenshotTestDefaultTestSuite.

- App (`com.makd.afinity`) retains
`android.permission.CHANGE_WIFI_MULTICAST_STATE` for mDNS discovery in
`LocalServiceDiscovery.kt` (which guards `WifiManager` access) while explicitly
setting `<uses-feature android:name="android.hardware.wifi"
android:required="false" />`, guarded by `AppManifestFeaturesRobolectricTest `.
- Full Gradle verification pipeline runs `./gradlew ktfmtCheck
checkCancellationRethrow checkItemProjection compileDebugKotlin
testDebugUnitTest :app:lintDebug :app:testDebugScreenshotTestDefaultTestSuite
:app:assembleDebug`.

- Play Store package `xyz.klinker.jellyfin` is published from Jenkins via
`fastlane android publish_release`, which builds `:app:bundleRelease`, uses
`app.versionName` from `gradle.properties` unmodified, and computes version
codes as `app.versionCode * 10000 + BUILD_NUMBER` (staying above existing Play
track version codes).
- Fastlane `publish_release` reads comma-separated `PLAY_TRACKS` (defaulting to
`internal,alpha` via `PlayReleaseHelper.parse_tracks`, replacing `PLAY_TRACK`),
uploading the `.aab` to the first track and promoting the release to remaining
tracks with status controlled by `PLAY_RELEASE_STATUS` (default `completed`, or
`draft` before initial Play review approval).
- Foreground service declarations: `FOREGROUND_SERVICE_MEDIA_PLAYBACK` is used
by Media3 `AudioService` for background Jellyfin and Audiobookshelf audio
playback, and `FOREGROUND_SERVICE_DATA_SYNC` is used by WorkManager workers
`MediaDownloadWorker` and `AbsMediaDownloadWorker` for user-initiated offline
media downloads.

- PlayerViewModel (ui/player/PlayerViewModel.kt) handles playback source
resolution across local offline files, DirectPlay, and automatic Transcode
fallback (retryWithTranscodeFallback / forceTranscodeFallback on decode errors),
using atomic StateFlow.update mutations for _uiState across Main and
Dispatchers.IO coroutines.
- Repository enforces ktfmtCheck via a Git pre-push hook and runs CI unit test
builds on Jenkins via unraid-server-android-agent on unraid-nas (192.168.1.95).

- In AudioService.kt, both ABS_AUDIO_ATTRIBUTES (audiobooks/podcasts) and
MUSIC_AUDIO_ATTRIBUTES intentionally use C.AUDIO_CONTENT_TYPE_MUSIC (alongside
MOVIE in PlayerViewModel and MUSIC in ThemeMusicPlayer) so media playback ducks
rather than pauses on incoming notifications.

- In TV mode (com.makd.afinity), HomeScreen directs initial focus to the first
available content row below HeroCarousel via HomeTvFocus.kt and
LocalCardRowInitialFocusRequester (prioritizing continue
watching/reading/listening, then next up, then library rows while waiting for
initial row loads) and scrolls the LazyColumn to (0, 0) when focus moves to
HeroCarousel.
- Detail screens (ItemDetailScreen.kt, DetailPrimaryButton.kt,
PlaybackSelectionButton.kt) and playback side sheets immediately move focus to
the primary Play button in TV mode, covered by TvFocusNavigationRobolectricTest.
- Focused TV UI elements use an animated scale-up indication (FocusIndication.kt
with TraversableNode handling for nested focusables, paired with
CardDimensions.kt and clip ordering adjustments) instead of focus outlines.

- Successful online playback syncs in `JellyfinPlaybackRepository` (non-music
`reportPlaybackProgress` and `reportPlaybackStop`) and `PodcastProgressTracker`
call `UserDataDao.updateSyncedPlaybackState` (`toBeSynced = 0`, preserving
already-played state) so `UserDataSyncWorker` cannot overwrite newer server
progress with stale offline `userdata` rows.
- `JellyfinAudiobookLoader` and `PodcastEpisodeDetailViewModel` overlay unsynced
local `userdata` (`toBeSynced = true`) onto online responses, and
`MusicTrackEntity.toAfinityTrack` preserves local `playbackPositionTicks` and
`played` state for downloaded podcasts.
- `JellyfinAudiobookReporter` schedules `UserDataSyncScheduler` when progress or
stop calls fail offline and retries `reportPlaybackStart` after offline starts
or pauses >= 15 minutes.

- Codebase is a single-module Android app (`:app`, package `com.makd.afinity`,
~250k LOC across ~1,070 Kotlin files) forked from `MakD/AFinity`, where major
multi-module or Kotlin Multiplatform restructures permanently break upstream
mergeability.
- Kotlin Multiplatform adoption is blocked across layers by heavy coupling to
Android `R.*` resources, Hilt (`@HiltViewModel`), `Timber`, and
`android.content.Context`, while pervasive `java.*` usage (`UUID`, `java.time`,
`java.io`, `java.net`) can be retained on JVM Desktop (Linux) but requires a
data-layer rewrite (`kotlin.uuid`, `kotlinx-datetime`, Okio) for Web (Wasm/JS).

- TV launcher UI components in `com.makd.afinity.tvlauncher.ui` (such as
`TvAppsRowSection.kt` and `TvAppActionsDialog`) use `LocalIsTelevision` and
`rememberTvInitialFocusRequester` for TV D-pad focus management, which must be
provided alongside `InputMode.Keyboard` in Robolectric TV UI tests
(`TvLauncherUiRobolectricTest`).
- The full Gradle verification suite runs `ktfmtCheck`,
`checkCancellationRethrow`, `checkItemProjection`, `compileDebugKotlin`,
`testDebugUnitTest`, `:app:lintDebug`, and
`:app:testDebugScreenshotTestDefaultTestSuite`.

- Optional TV launcher mode is isolated under
`app/src/main/java/com/makd/afinity/tvlauncher/` (`TvLauncherManager`,
`TvLauncherStatus`, `TvInstalledAppsRepository`, `TvAppArtworkCache`,
`TvLauncherViewModel`, and `ui/` composables) backed by a disabled-by-default
`.TvLauncherAlias` `<activity-alias>` in `AndroidManifest.xml` and lazy
`TvLauncherManager` injection on non-TV devices.
- When TV launcher mode is active, TV Settings shows the `adb` command to set
the default home activity, the Home screen adds an installed apps row (excluding
`com.makd.afinity` and evicting `TvAppArtworkCache` entries on `LauncherApps`
callbacks), login/unauthenticated screens render `TvLauncherFallbackPanel`
(respecting `LocalPinnedDrawerStartInset`), and `TvLauncherManager.disableAlias`
blocks turning off the alias if no other enabled `CATEGORY_HOME` launcher
exists.
- Full Gradle verification runs `ktfmtCheck`, `checkCancellationRethrow`,
`checkItemProjection`, `compileDebugKotlin`, `testDebugUnitTest` (using
`@RunWith(RobolectricTestRunner::class)` for `PackageManager` and Compose UI
tests), `:app:lintDebug`, and `:app:testDebugScreenshotTestDefaultTestSuite`.
