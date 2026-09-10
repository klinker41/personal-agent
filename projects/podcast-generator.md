---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-10T00:33:07.806584+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- Single-Container Runtime: Consolidated 6 legacy microservices (PostgreSQL,
  Redis, Celery, FastAPI, Nginx) into an `oven/bun:alpine` container (Bun, Hono,
  FFmpeg) running React SPA, REST APIs, media streams, and movie generation
  (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`).
- Minimal Dependencies: Standardized on minimal backend packages (`hono`,
  `jsonwebtoken`, `cron-parser@4.9.0`) using native `hono/cors`, `Bun.password`,
  and native test execution (`bun test`). Replaced database dependencies (`pg`,
  migrations) with filesystem JSON persistence (`data/users.json`,
  `data/podcasts/`, `data/outputs/`), an in-process concurrency-limited
  `JobQueue`, and `PodcastScheduler`.
- Storage Utilities & Security: `fileStore.ts` provides atomic file writes
  (`writeText`, `writeJson`), folder validation, date matching, and Unicode NFC
  normalization preserving `[\p{L}\p{N}\p{M}]` for diacritics. Audio and cover
  streaming endpoints enforce `path.sep` boundaries and non-blocking
  `Bun.file(path).exists()` checks to prevent path traversal. Unified streaming
  and authorization helpers across backend routes.
- Manifests & Discovery: `EpisodeStore.autoDiscoverEpisodes` backfills missing
  `episode.json` manifests on `GET /api/podcasts/:id` (omitted from
  `GET /api/v1/podcasts/` to prevent latency bottlenecks). Podcast manifests
  support `ad_reads?: string[] | null`, managed via dynamic textareas in
  `PodcastForm.tsx` and handled in `podcasts.ts` CRUD routes.

## Media Pipeline, Resilience & Deduplication
- Media Generation & Ad Insertion: Synthesizes episodes via Gemini (script, 1:1
  cover art, multi-speaker TTS) and FFmpeg ID3v2 tagging. Episode prompt
  generation (`generator.ts`) uses `formatAdReads()` to position N ad reads at
  evenly spaced fractional intervals `k / (N + 1)` across the episode.
- Lib
<truncated 72 bytes>
nd/src/services/generator.ts`; copies finished MP3s and appends
  `<track>` metadata to `album.nfo` via `EXTERNAL_OUTPUTS_DIR`.
- Error Handling & Resilience: Centralized HTTP exponential backoff in
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
- Auth Hardening: Startup fails fast in production if `JWT_SECRET` is unset or
  uses default placeholders in `backend/src/middleware/authMiddleware.ts`.
  Supports `ALLOW_REGISTRATION` env var and `GET /api/v1/auth/config` to disable
  public registration and toggle UI links.
- Security Hygiene: Scrub git history (`git-filter-repo` or squashed initial
  commit) prior to public release to purge leaked Gemini API keys, Jellyfin
  tokens, and private domain references.
- Test Mocking: Unit tests executing generator or error notification workflows
  must mock `NotificationManager.prototype.notify` to avoid dispatching live
  Slack webhook requests when `SLACK_WEBHOOK_URL` is configured.
