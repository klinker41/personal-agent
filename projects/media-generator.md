---
topic: media-generator
category: projects
tags: [project, media-generator, movie-generator, podcast-generator, bun, hono]
updated_at: 2026-10-30T18:59:00+00:00
compacted_at: 2026-10-30T18:59:00+00:00
confidence: 0.95
---

# Project: Media-Generator (Media Studio)

## Model Configuration & Selection
- **Centralized Model IDs & Selection:** Default generative model IDs across all
  media categories (text, image, audio, video, music) are centralized as
  exported constants in `shared/models.ts` and `.env.example` without `models/`
  prefixes: `DEFAULT_TEXT_MODEL` (`gemini-3.8-flash`), `DEFAULT_IMAGE_MODEL`
  (`gemini-3-pro-image`), `DEFAULT_AUDIO_MODEL` (`gemini-3.8-flash-tts`),
  `DEFAULT_VIDEO_MODEL` (`gemini-omni-1.1-flash`), and `DEFAULT_MUSIC_MODEL`
  (`lyria-3-clip-preview`). Model selection permits preview models matching
  `gemini-*` or `lyria-*`, but strictly excludes experimental (`exp`,
  `experimental`) and alias (`latest`) models in favor of stable or preview
  releases.

## Architecture, Concurrency & Storage
- **Runtime Stack & Routing:** Built on `oven/bun:alpine` with Bun, Hono, and
  CLI `ffmpeg`, serving a React SPA, REST APIs, audio streaming, and movie
  pipelines (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`). Dependencies
  are kept minimal (`jsonwebtoken`, `cron-parser@4.9.0`), using `hono/cors`,
  `Bun.password`, and `bun test` (`app.request()`). Route `/assets/*` guards
  against directory traversal, serving `generated_assets/` with fallback to
  `frontend/dist/assets/`.
- **Task Queues & Crash Recovery:** `JobQueue`, `AudiobookQueue`, and
  `BookQueue` inherit from `BaseTaskQueue` with re-entrancy guards to enforce
  FIFO gating, alongside `PodcastScheduler`. Server startup executes
  `BookStore.recoverStuckM4bBuilds()` to automatically reconcile and resume
  interrupted audiobook compilation tasks stuck in `m4b_status: building`.
- **Tenancy, Persistence & Safe Writes:** Isolated via `data/users.json`,
  `data/podcasts/`, and `data/outputs/` (`GET /api/podcasts` resolves tenancy
  via `PodcastStore.list(user.id)`). File stores (`EpisodeStore`, `UserStore`,
  `fileStore.ts`) and audiobook routes validate ID path segments, enforce
  directory containment with `path.sep` boundaries, and use atomic writes
  (`writeText`, `writeJson`). `EpisodeStore.autoDiscoverEpisodes` lazily
  backfills missing `episode.json` on `GET /api/podcasts/:id` (omitted from
  listings to prevent latency).

## Video Generation & Continuity
- **Pipeline & Tiers:** Orchestrated by `gemini-3.7-flash` with video generation
  via `gemini-omni-1.1-flash`. Stitches chunks into `scene.mp4` and `movie.mp4`
  across 4 tiers (360p Draft, 720p HD, 1080p FHD, 4K UHD), skipping existing or
  upscaled chunks.
- **Shot Continuity & Plates:** Stage 4.8A creates concept plates with
  `generateCameraSetupImagePrompt`. In `backend/src/services/gemini.ts`,
  `generateSceneChunks` branches on `camera_continuity` (`continuous` vs
  `new_shot`), attaching prior `video.mp4` with temporal cues as multimodal
  references for continuous shots.
- **Safety, Diagnostics & Serialization:** Sets `BLOCK_ONLY_HIGH` across all
  categories in `DEFAULT_SAFETY_SETTINGS` (including
  `HARM_CATEGORY_CIVIC_INTEGRITY`). Blocked prompts or `PROHIBITED_CONTENT`
  trigger Flash sanitization (`sanitizeSceneContent` / `sanitizeAndFixPrompt`),
  saving updated `setting` and `action_summary` to `prompt.txt` and
  `chunk_manifest.json`. In `backend/src/utils/jsonParser.ts`,
  `extractGeminiResponseText` checks `finishReason`, `safetyRatings`, and
  `blockReason` on empty outputs, while `formatTimelineBeat` and
  `formatTimelineAndAudio` explicitly format timeline beats to prevent
  `[object Object]` serialization.

## Podcast Pipeline & Integrations
- **Generation & Ad Insertion:** Gemini (`TEXT_MODEL`, `IMAGE_MODEL`,
  `AUDIO_MODEL`) produces scripts, 1:1 cover art, and multi-speaker TTS with
  FFmpeg ID3v2 tags. `formatAdReads()` (`generator.ts`) evenly spaces ad reads
  (`ad_reads?: string[] | null` via `PodcastForm.tsx` / `podcasts.ts`) at
  `k / (N + 1)` intervals.
- **Deduplication & Matching:** `EpisodeStore.deduplicate()`
  (`backend/src/index.ts`) drops duplicate paths and titles, favoring longer
  scripts and records with `prompt_used`. `buildTitlePatterns` compiles exact,
  Unicode NFC (`[\p{L}\p{N}\p{M}]`), and stripped ASCII regexes, guarding
  against empty strings.
- **Jellyfin Sync:** Copies MP3s to `EXTERNAL_OUTPUTS_DIR`, maps disc numbers
  via `TITLE_TO_DISC_MAPPING`, appends `<track>` entries to `album.nfo`, and
  triggers Jellyfin sync (`POST /Items/{itemId}`) using an admin discovered from
  `GET /Users`.

## Security, Reliability & Testing
- **Auth & RBAC:** Startup halts if `JWT_SECRET` is unset or default
  (`authMiddleware.ts`). Authentication redirects to `/` instead of a dedicated
  `/login` route, managing first-time admin setup completion via `App.tsx` state
  with a single internal `AuthProvider`. `ALLOW_REGISTRATION` governs signups
  (`GET /api/v1/auth/config`); boundary checks enforce `created_by === user.id`,
  and admin rights strictly require both `"role": "admin"` and
  `"is_admin": true` in `data/users.json`. Streaming validates `path.sep`
  boundaries via non-blocking `Bun.file(path).exists()`.
- **Retries & Error Handling:** `postWithRetry` (`gemini.ts`) applies
  exponential backoff; queue failures invoke `failJob` (`queue.ts`).
  `GeminiService.generateAudioPart` retries up to 3 times on non-200 responses
  or content filtering (returns `false` without throwing), aborting if
  `failedParts >= 5`.
- **Alerts & Test Isolation:** Slack alerts (`SLACK_WEBHOOK` or
  `SLACK_WEBHOOK_URL`, sender `podcast-generator`) build canonical RSS feed URLs
  with `BASE_URL` or fallback port. Notification and generation tests must mock
  `NotificationManager.prototype.notify` to prevent live dispatches.
