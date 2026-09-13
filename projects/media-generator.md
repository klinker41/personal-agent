---
topic: media-generator
category: project
tags: [project, media-generator]
updated_at: 2026-09-13T00:34:23.495859+00:00
confidence: 0.95
---

# Project: Media-Generator

## Platform & Architecture
- **Branding & Stack**: Branded as 'Media Studio' in the top app bar (not
  'AI Media Studio' or 'Podcast Generator'). Implemented as a unified Bun and
  Hono web service with JWT authentication, uniting cinematic movie
  generation, episodic podcast synthesis, and audiobook creation.
- **RBAC & User Management**: Multi-user role-based access control requires an
  admin role in users.json on disk to access and manage legacy creations.
- **Workspace Tooling**: Root scripts coordinate build, lint, typecheck, and
  test across workspaces using Bun and Vite. Backend tests run via `bun test`;
  frontend tests run via `vitest run`.

## Media Pipelines
- **Cinematic Movie Pipeline**:
  - 7-stage workflow: Prompting, plot formulation, screenplay breakdown,
    character casting with reference portraits, scene chunking with camera
    setups/plates, Gemini Omni video generation with temporal continuity, and
    FFmpeg stitching with single-chunk regeneration and upscaling.
  - Camera continuity: In `movieGemini.ts`, scene chunking enforces
    shot/reverse-shot rules where speaker alternations mandate `new_shot`.
    Continuous shots (`camera_continuity: 'continuous'`) are reserved for
    extended single-speaker dialogue exceeding chunk limits (10s or 18–20
    words) or sustained shared staging.
  - Video stitching: FFmpeg concat demuxer lists require quote escaping,
    unique temporary file paths, and re-encode fallback handling for mismatched
    stream parameters across chunks.
- **Episodic Podcast Pipeline**:
  - Multi-speaker synthesis: Dialogue synthesized via Gemini TTS with celestial
    voice profiles, ID3v2-tagged MP3 mastering, automated cron releases, and
    RSS 2.0 feeds with iTunes tags.
  - Speaker guidelines: In `formatSpeakerGuidelines`, one host must always
    announce the podcast name at the start regardles
<truncated 3512 bytes>
tching to an offline
  fallback user) and only purges tokens upon explicit HTTP 401 or 403
  responses, supporting both `token` and `auth_token` keys. Handles clean
  EventSource SSE stream termination on unmount and disconnect.
- **Dashboard Layout**:
  - Desktop (`lg:`): Asymmetric 2-column layout (65% creations feed, 35%
    operations sidebar).
  - Mobile (`<sm`): Collapses to a single column with a 2x2 telemetry grid.
  - Components: KPI cards (`DashboardCards.tsx`), an in-flight pipeline banner
    (`InFlightPipelineBanner.tsx`) with visualizer links, and creations feed
    (`MediaCreationsFeed.tsx`). Reference screenshots stored in `docs/images/`
    (`dashboard-desktop.jpg`, `dashboard-mobile.jpg`).
- **Podcasts List UI**: Mobile layout replaces nested container padding
  (`max-w-7xl px-4 py-8`) with `space-y-6 w-full`, using responsive cards with
  top-right status toggles, metadata badges, and expanded bottom action footers.
- **Form Validation & State Immutability**:
  - `PodcastForm` validates against empty or whitespace-only titles across all
    submit triggers prior to invoking backend APIs.
  - `StepInspector` preserves prop immutability during prompt editing,
    resolution switching, and video regeneration by dispatching object copies
    rather than mutating props in place.
- **UI Cleanup**: Pruned duplicate `Dashboard.tsx` and legacy `LoginPage.tsx`,
  migrating test coverage to `pages/MoviesList.tsx`.

## Performance & Testing
- **Performance Optimizations**:
  - Replaced $O(N \times M)$ per-podcast disk scans in `PodcastStore.list()`
    with a single-pass active generation `Set`.
  - Added a 5-second mutation-invalidated in-memory cache to `EpisodeStore`.
- **Test Coverage & Known Issues**:
  - Critical test coverage gaps exist in `JobQueue`
    (`backend/src/services/queue.ts`) and startup cleanup
    `EpisodeStore.deduplicate()` (`backend/src/services/fileStore.ts`).
  - Frontend Vitest execution logs unhandled `ERR_INVALID_URL` warnings due to
    unmocked fetch calls in `setupTests.ts`.
