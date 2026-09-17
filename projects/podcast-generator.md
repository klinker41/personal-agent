---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-17T00:33:10.138910+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- **Single-Container Runtime**: Consolidated 6 legacy microservices
  (PostgreSQL, Redis, Celery, FastAPI, Nginx) into an `oven/bun:alpine`
  container running Bun, Hono, and FFmpeg for the React SPA, REST APIs,
  media streaming, and movie generation (`movieGemini.ts`, `moviePipeline.ts`,
  `MovieStore`). Standardized on minimal dependencies (`hono`, `jsonwebtoken`,
  `cron-parser@4.9.0`), native `hono/cors`, `Bun.password`, and `bun test`.
- **Filesystem Persistence & Manifests**: Replaced databases with JSON storage
  (`data/users.json`, `data/podcasts/`, `data/outputs/`), an in-process
  concurrency-limited `JobQueue`, and `PodcastScheduler`. Stamped manifests
  enforce per-user tenancy (`GET /api/podcasts` filters via
  `PodcastStore.list(user.id)`; episodes inherit tenancy) and support optional
  `ad_reads?: string[] | null` managed via `PodcastForm.tsx` and `podcasts.ts`.
  `EpisodeStore.autoDiscoverEpisodes` backfills missing `episode.json` on
  `GET /api/podcasts/:id` (omitted from listings to prevent latency).
- **File Utilities & Security**: `fileStore.ts` provides atomic file operations
  (`writeText`, `writeJson`), folder validation, date matching, and Unicode NFC
  normalization preserving `[\p{L}\p{N}\p{M}]` for diacritics. Media streaming
  enforces `path.sep` boundaries and non-blocking `Bun.file(path).exists()`
  checks against path traversal. Unified streaming and route auth helpers.

## Media Pipeline, Integrations & Resilience
- **Generation & Library Sync**: Synthesizes episodes via Gemini (script, 1:1
  cover art, multi-speaker TTS) and FFmpeg ID3v2 tagging. Distributes N ad reads
  evenly at fractional intervals `k / (N + 1)` across dialogue via
  `formatAdReads()` in `generator.ts`. Resolves disc numbers via
  `TITLE_TO_DISC_MAPPING`, copies finished MP3s and appends `<track>` metadata
  to `album.nfo` in `EXTERNAL_OUTPUTS_DIR`, and updates Jellyfin item metadata
  (`POST /Items/{itemId}`) using an admin user ID resolved from `GET /Users`.
- **Resilience & Retry Strategy**: HTTP requests use centralized exponential
  backoff via `postWithRetry` (`gemini.ts`); queue errors standardize on
  `failJob` in `queue.ts`. `GeminiService.generateAudioPart` retries up to 3
  times on non-200 responses or API filtering (returning false instead of
  throwing). `PodcastGenerator.generateAudio` aborts only after 5 failed audio
  chunks (`failedParts >= 5`).
- **Deduplication & Title Matching**: `EpisodeStore.deduplicate()` in
  `backend/src/index.ts` prunes duplicate records by audio path or title
  (preferring entries with `prompt_used` and longer scripts).
  `buildTitlePatterns` pre-compiles multi-strategy regexes (exact, Unicode NFC,
  legacy stripped ASCII), guarding against empty regexes from unsafe characters.

## Configuration, Security & Testing
- **Configuration & Alerts**: AI models (`TEXT_MODEL`, `IMAGE_MODEL`,
  `AUDIO_MODEL`) and Slack alerts (`SLACK_WEBHOOK` or `SLACK_WEBHOOK_URL`,
  default username `podcast-generator`) configure via environment variables
  (legacy Ntfy and ComfyUI removed). Completion alerts include canonical RSS
  feed URLs from `BASE_URL` or fallback port; settings UI is restricted to
  account/security.
- **Auth Hardening & Git Hygiene**: Startup fails fast in production if
  `JWT_SECRET` is unset or uses default placeholders in `authMiddleware.ts`.
  Public registration can be disabled via `ALLOW_REGISTRATION` env var and
  queried via `GET /api/v1/auth/config`. Scrub git history (`git-filter-repo` or
  squashed initial commit) prior to release to purge leaked Gemini API keys,
  Jellyfin tokens, and private domain references.
- **Test Mocking**: Unit tests executing generator or error notification
  workflows must mock `NotificationManager.prototype.notify` to avoid
  dispatching live Slack webhook requests when `SLACK_WEBHOOK_URL` is
  configured.
