---
topic: media-generator
category: project
tags: [project, media-generator]
updated_at: 2026-09-12T00:33:58.350203+00:00
confidence: 0.95
---

# Project: Media-Generator

## Platform & Architecture
- **Branding & Stack**: Branded as 'Media Studio' in the top app bar (not
  'AI Media Studio' or 'Podcast Generator'). Implemented as a unified Bun
  and Hono web service with JWT authentication, uniting cinematic movie
  generation, episodic podcast synthesis, and audiobook creation.
- **Workspace Tooling**: Root scripts coordinate build, lint, typecheck, and
  test across workspaces using Bun and Vite. Backend tests run via `bun test`;
  frontend tests run via `vitest run`.

## Media Pipelines
- **Cinematic Movie Pipeline**: 7-stage workflow spanning prompting, plot
  formulation, screenplay breakdown, character casting with reference
  portraits, scene chunking with camera setups/plates, Gemini Omni video
  generation with temporal continuity, and FFmpeg stitching with single-chunk
  regeneration and upscaling. In `movieGemini.ts`, scene chunking enforces
  shot/reverse-shot continuity where speaker alternations mandate `new_shot`,
  making continuous camera shots rare outside extended single-speaker dialogue
  exceeding chunk limits.
- **Episodic Podcast Pipeline**:
  - Multi-Speaker Synthesis: Synthesizes dialogue via Gemini TTS with celestial
    voice profiles, ID3v2-tagged MP3 mastering, automated cron releases, and
    RSS 2.0 feeds with iTunes tags.
  - Speaker Guidelines: In `formatSpeakerGuidelines`, one host must always
    announce the podcast name at the start regardless of banter setting;
    banter-off mode begins discussion promptly without small talk.
  - Ad Reads: Supports sponsor segments via the `ad_reads` property on
    `Podcast`, distributing segments evenly across episode dialogue.
  - Theme Music: Uses Google Lyria 3 Clip (`lyria-3-clip-preview`, overridable
    via `MUSIC_MODEL` or `GEMINI_MUSIC_MODEL`). Audio files are stored in
    `data/podcasts/music/` (`music_{intro|outro}_<timestamp>_<uuid>
<truncated 4525 bytes>
`AuthContext` preserves sessions during network
  outages and 5xx errors using decoded JWT data (switching to an offline
  fallback user) and only purges tokens upon explicit HTTP 401 or 403
  responses, supporting both `token` and `auth_token` keys.
- **Dashboard Layout**:
  - Desktop (`lg:`): Asymmetric 2-column layout (65% creations feed, 35%
    operations sidebar).
  - Mobile (`<sm`): Collapses to a single column with a 2x2 telemetry grid.
  - Components: KPI cards (`DashboardCards.tsx`), an in-flight banner
    (`InFlightPipelineBanner.tsx`) with visualizer links, and creations feed
    (`MediaCreationsFeed.tsx`). Reference screenshots stored in `docs/images/`
    (`dashboard-desktop.jpg`, `dashboard-mobile.jpg`).
- **Podcasts List UI**: Mobile layout replaces nested container padding
  (`max-w-7xl px-4 py-8`) with `space-y-6 w-full`, using responsive cards with
  top-right status toggles, metadata badges, and expanded bottom action footers.
- **Form Validation & State**:
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
