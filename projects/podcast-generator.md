---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-20T00:32:18.618999+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- **Single-Container Runtime**: Consolidated 6 legacy microservices
  (Postgres, Redis, Celery, FastAPI, Nginx) into an `oven/bun:alpine`
  container running Bun, Hono, and FFmpeg for the React SPA, REST APIs,
  streaming, and movie generation (`movieGemini.ts`, `moviePipeline.ts`,
  `MovieStore`). Uses minimal dependencies: `hono`, `jsonwebtoken`,
  `cron-parser@4.9.0`, native `hono/cors`, `Bun.password`, and `bun test`.
- **Filesystem Persistence & Tenancy**: Filesystem JSON manifests
  (`data/users.json`, `data/podcasts/`, `data/outputs/`) enforce per-user
  tenancy (`PodcastStore.list(user.id)` on `GET /api/podcasts`; episodes
  inherit tenancy). Managed by in-process concurrency-limited `JobQueue` and
  `PodcastScheduler`. `fileStore.ts` provides atomic writes (`writeText`,
  `writeJson`), directory validation, and date matching.
  `EpisodeStore.autoDiscoverEpisodes` backfills missing `episode.json` on
  `GET /api/podcasts/:id` (omitted from list endpoints to prevent latency).

## Media Pipeline & Integrations
- **Generation & Ad Insertion**: Synthesizes episodes via Gemini (script, 1:1
  cover art, multi-speaker TTS) and FFmpeg ID3v2 tagging. Distributes optional
  ad reads (`ad_reads?: string[] | null`, configured via `PodcastForm.tsx` and
  `podcasts.ts`) evenly across dialogue at fractional intervals `k / (N + 1)`
  via `formatAdReads()` in `generator.ts`.
- **Deduplication & Matching**: `EpisodeStore.deduplicate()` in
  `backend/src/index.ts` prunes duplicate records by audio path or title
  (preferring entries with `prompt_used` and longer scripts).
  `buildTitlePatterns` pre-compiles regexes (exact, Unicode NFC preserving
  `[\p{L}\p{N}\p{M}]` for diacritics, stripped ASCII), guarding against empty
  regexes from unsafe characters.
- **Library Sync & Metadata**: Copies finished MP3s, maps disc numbers via
  `TITLE_TO_DISC_MAPPING`, appends `<track>` metadata to `album.nfo` in
  `EXTERNAL_OUTPUTS_DIR`, and updates Jellyfin item metadata
  (`POST /Items/{itemId}`) using an admin user ID resolved from `GET /Users`.

## Resilience, Security & Testing
- **Auth Hardening & Git Hygiene**: Startup fails fast in production if
  `JWT_SECRET` is unset or default in `authMiddleware.ts`. Public registration
  can be disabled via `ALLOW_REGISTRATION` (`GET /api/v1/auth/config`). Media
  streaming enforces `path.sep` boundaries and non-blocking
  `Bun.file(path).exists()` checks against traversal. Git history must be
  purged (`git-filter-repo` or squashed commit) before release to remove
  leaked secrets (Gemini keys, Jellyfin tokens, private domains).
- **Retries & Error Handling**: HTTP requests use centralized exponential
  backoff via `postWithRetry` (`gemini.ts`); queue errors standardize on
  `failJob` (`queue.ts`). `GeminiService.generateAudioPart` retries up to 3
  times on non-200 responses or API filtering (returns false without throwing).
  `PodcastGenerator.generateAudio` aborts only when `failedParts >= 5`.
- **Configuration, Alerts & Testing**: Environment variables configure AI
  models (`TEXT_MODEL`, `IMAGE_MODEL`, `AUDIO_MODEL`) and Slack alerts
  (`SLACK_WEBHOOK` or `SLACK_WEBHOOK_URL`, default user `podcast-generator`;
  legacy Ntfy and ComfyUI removed). Completion alerts include canonical RSS
  feed URLs from `BASE_URL` or fallback port; settings UI is restricted to
  account/security. Unit tests for generator or notifications must mock
  `NotificationManager.prototype.notify` to avoid live Slack webhooks.
