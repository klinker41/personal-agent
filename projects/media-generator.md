---
topic: media-generator
category: project
tags: [project, media-generator]
updated_at: 2026-09-23T00:33:45.209375+00:00
confidence: 0.95
---

# Project: Media-Generator

## Platform & Architecture
- **Branding & Stack**: Branded as 'Media Studio' in the top app bar (not
  'AI Media Studio' or 'Podcast Generator'). Built as a unified Bun and Hono
  web service with JWT authentication, uniting cinematic movie generation,
  episodic podcast synthesis, and audiobook creation.
- **RBAC & User Management**: Multi-user role-based access control requires an
  admin role in `users.json` on disk to access and manage legacy creations;
  standard users can view only their own creations.
- **Workspace Tooling**: Root scripts coordinate build, lint, typecheck, and
  testing across workspaces using Bun and Vite (`bun test` for backend,
  `vitest run` for frontend).

## Model Configuration & Standards
- **Centralized Model Defaults**: Centralized in `shared/models.ts` and
  mirrored in `.env.example` across five constants without `models/` prefix:
  `DEFAULT_TEXT_MODEL` (`gemini-3.8-flash`), `DEFAULT_IMAGE_MODEL`
  (`gemini-3-pro-image`), `DEFAULT_AUDIO_MODEL`
  (`gemini-3.1-flash-tts-preview`), `DEFAULT_VIDEO_MODEL`
  (`gemini-omni-1.1-flash`), and `DEFAULT_MUSIC_MODEL`
  (`lyria-3-clip-preview`).
- **Model Selection & Update Policies**: Model names must strip `models/`
  prefixes and use non-experimental Gemini models (Lyria for music), strictly
  excluding experimental tags (`exp`, `experimental`, `latest`) while permitting
  preview releases. Tier rules mandate Flash for text, audio, and video, and Pro
  for images. Automated model updates via sidecar commit locally and notify via
  Slack; pushing requires explicit user approval.

## Media Pipelines & Storage
- **Cinematic Movie Pipeline**: 7-stage workflow (prompting, plot formulation,
  screenplay breakdown, character casting with portraits, scene chunking with
  plates, Gemini Omni video with temporal continuity, FFmpeg stitching with
  single-chunk regeneration and up
<truncated 1340 bytes>
essions
  during network outages and 5xx errors using decoded JWT data (switching to an
  offline fallback user) and only purges tokens upon explicit HTTP 401 or 403
  responses, supporting both `token` and `auth_token` keys. EventSource SSE
  streams terminate cleanly on unmount and disconnect.
- **Dashboard Layout**: Desktop (`lg:`): Asymmetric 2-column layout (65%
  creations feed, 35% operations sidebar); mobile (`<sm`): Single column with a
  2x2 telemetry grid. Components: KPI cards (`DashboardCards.tsx`), in-flight
  pipeline banner (`InFlightPipelineBanner.tsx`) with visualizer links, and
  creations feed (`MediaCreationsFeed.tsx`). Reference screenshots reside in
  `docs/images/` (`dashboard-desktop.jpg`, `dashboard-mobile.jpg`).
- **Podcasts List UI**: Mobile layout replaces nested container padding
  (`max-w-7xl px-4 py-8`) with `space-y-6 w-full`, using responsive cards with
  top-right status toggles, metadata badges, and expanded bottom action footers.
- **Form Validation & State Immutability**: `PodcastForm` validates against
  empty or whitespace-only titles across all submit triggers prior to invoking
  backend APIs. `StepInspector` preserves prop immutability during prompt
  editing, resolution switching, and video regeneration by dispatching object
  copies rather than mutating props in place.
- **UI Cleanup**: Pruned duplicate `Dashboard.tsx` and legacy `LoginPage.tsx`,
  migrating test coverage to `pages/MoviesList.tsx`.

## Performance & Testing
- **Performance Optimizations**: Replaced $O(N \times M)$ per-podcast disk
  scans in `PodcastStore.list()` with a single-pass active generation `Set`.
  Added a 5-second mutation-invalidated in-memory cache to `EpisodeStore`.
- **Test Coverage & Known Issues**: Critical test coverage gaps exist in
  `JobQueue` (`backend/src/services/queue.ts`) and startup cleanup
  `EpisodeStore.deduplicate()` (`backend/src/services/fileStore.ts`). Frontend
  Vitest execution logs unhandled `ERR_INVALID_URL` warnings due to unmocked
  fetch calls in `setupTests.ts`.
