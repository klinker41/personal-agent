---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-12T00:32:44.996324+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- Single-Container Runtime: Consolidated 6 legacy microservices (PostgreSQL,
  Redis, Celery, FastAPI, Nginx) into an `oven/bun:alpine` container
  (Bun, Hono, FFmpeg) running React SPA, REST APIs, media streams, and movie
  generation (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`). Standardized
  on minimal dependencies (`hono`, `jsonwebtoken`, `cron-parser@4.9.0`), native
  `hono/cors`, `Bun.password`, and native test execution (`bun test`).
- Filesystem Persistence & Tenancy: Replaced database dependencies (`pg`,
  migrations) with filesystem JSON storage (`data/users.json`, `data/podcasts/`,
  `data/outputs/`), an in-process concurrency-limited `JobQueue`, and
  `PodcastScheduler`. Stamped podcast manifests enforce per-user tenancy via
  `user_id` (`GET /api/podcasts` filters via `PodcastStore.list(user.id)`, and
  episodes inherit tenancy from parent podcasts). Manifests support
  `ad_reads?: string[] | null` configured via dynamic textareas in
  `PodcastForm.tsx` and handled in `podcasts.ts`.
  `EpisodeStore.autoDiscoverEpisodes` backfills missing `episode.json` on
  `GET /api/podcasts/:id` (omitted from `GET /api/v1/podcasts/` to eliminate
  listing latency bottlenecks).
- Storage Utilities & Streaming Security: `fileStore.ts` provides atomic file
  writes (`writeText`, `writeJson`), folder validation, date matching, and
  Unicode NFC normalization preserving `[\p{L}\p{N}\p{M}]` for diacritics. Audio
  and cover streaming endpoints enforce `path.sep` boundaries and non-blocking
  `Bun.file(path).exists()` checks against path traversal. Unified streaming and
  authorization helpers across backend routes.

## Media Pipeline, Resilience & Integrations
- Media Generation & Ad Distribution: Synthesizes episodes via Gemini (script,
  1:1 cover art, multi-speaker TTS) and FFmpeg ID3v2 tagging. Distributes 
<truncated 432 bytes>
ative
  user ID and supplying it as the `userId` parameter on `GET /Items/{itemId}`
  before updating via `POST /Items/{itemId}`.
- Error Handling & Audio Resilience: Centralized HTTP exponential backoff in
  `postWithRetry` (`gemini.ts`) and standardized queue errors via `failJob` in
  `queue.ts`. `GeminiService.generateAudioPart` retries up to 3 times with
  backoff on non-200 responses or API filtering (returning false instead of
  throwing). `PodcastGenerator.generateAudio` aborts only after 5 failed chunks
  (`failedParts >= 5`).
- Deduplication & Title Matching: `EpisodeStore.deduplicate()` in
  `backend/src/index.ts` prunes disk and memory duplicate records by audio path
  or title (preferring entries with `prompt_used` and longer scripts).
  Pre-compiles multi-strategy regexes (exact, Unicode NFC, legacy stripped
  ASCII) via `buildTitlePatterns`, guarding against empty regex patterns from
  unsafe characters.

## Configuration, Security & Testing
- Configuration & Alerts: AI models (`TEXT_MODEL`, `IMAGE_MODEL`, `AUDIO_MODEL`)
  and Slack alerts (`SLACK_WEBHOOK` or `SLACK_WEBHOOK_URL`, default username
  `podcast-generator`) configure via environment variables; removed legacy Ntfy
  and ComfyUI. Episode completion alerts include canonical RSS feed URLs from
  `BASE_URL` or fallback port. Settings UI is restricted to account/security.
- Auth Hardening & Hygiene: Startup fails fast in production if `JWT_SECRET` is
  unset or uses default placeholders in
  `backend/src/middleware/authMiddleware.ts`. Supports `ALLOW_REGISTRATION` env
  var and `GET /api/v1/auth/config` to disable public registration and toggle UI
  links. Scrub git history (`git-filter-repo` or squashed initial commit) prior
  to public release to purge leaked Gemini API keys, Jellyfin tokens, and
  private domain references.
- Test Mocking: Unit tests executing generator or error notification workflows
  must mock `NotificationManager.prototype.notify` to avoid dispatching live
  Slack webhook requests when `SLACK_WEBHOOK_URL` is configured.
