---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-15T00:33:50.308939+00:00
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
- **Filesystem Persistence & Tenancy**: Replaced database dependencies (`pg`,
  migrations) with filesystem JSON storage (`data/users.json`,
  `data/podcasts/`, `data/outputs/`), an in-process concurrency-limited
  `JobQueue`, and `PodcastScheduler`. Stamped manifests enforce tenancy via
  `user_id` (`GET /api/podcasts` filters via `PodcastStore.list(user.id)`;
  episodes inherit tenancy) and support `ad_reads?: string[] | null` via
  `PodcastForm.tsx` and `podcasts.ts`. `EpisodeStore.autoDiscoverEpisodes`
  backfills missing `episode.json` on `GET /api/podcasts/:id` (omitted from
  listing routes to eliminate latency).
- **File Utilities & Security**: `fileStore.ts` provides atomic file writes
  (`writeText`, `writeJson`), folder validation, date matching, and Unicode NFC
  normalization preserving `[\p{L}\p{N}\p{M}]` for diacritics. Audio and cover
  streaming endpoints enforce `path.sep` boundaries and non-blocking
  `Bun.file(path).exists()` checks against path traversal. Unified streaming
  and route authorization helpers.

## Media Pipeline, Integrations & Resilience
- **Generation, Ads & Library Sync**: Synthesizes episodes via Gemini (script,
  1:1 cover art, multi-speaker TTS) and FFmpeg ID3v2 tagging. Distributes N ad
  reads evenly at fractional intervals `k / (N + 1)` across dialogue via
  `formatAdReads()` in `generator.ts`. Resolves disc numbers via title disc

<truncated 41 bytes>
ck>` metadata to `album.nfo` in
  `EXTERNAL_OUTPUTS_DIR`, and updates Jellyfin item metadata
  (`POST /Items/{itemId}`) using an admin user ID resolved from `GET /Users`.
- **Error Handling & Resilience**: Centralized HTTP exponential backoff in
  `postWithRetry` (`gemini.ts`) and standardized queue errors via `failJob` in
  `queue.ts`. `GeminiService.generateAudioPart` retries up to 3 times on
  non-200 responses or API filtering (returning false rather than throwing).
  `PodcastGenerator.generateAudio` aborts only after 5 failed chunks
  (`failedParts >= 5`).
- **Deduplication & Title Matching**: `EpisodeStore.deduplicate()` in
  `backend/src/index.ts` prunes disk and memory duplicate records by audio path
  or title (preferring entries with `prompt_used` and longer scripts).
  Pre-compiles multi-strategy regexes (exact, Unicode NFC, legacy stripped
  ASCII) via `buildTitlePatterns`, guarding against empty regex patterns from
  unsafe characters.

## Configuration, Security & Testing
- **Configuration & Alerts**: AI models (`TEXT_MODEL`, `IMAGE_MODEL`,
  `AUDIO_MODEL`) and Slack alerts (`SLACK_WEBHOOK` or `SLACK_WEBHOOK_URL`,
  default username `podcast-generator`) configure via environment variables
  (removed legacy Ntfy and ComfyUI). Completion alerts include canonical RSS
  feed URLs from `BASE_URL` or fallback port; settings UI is restricted to
  account/security.
- **Auth Hardening & Hygiene**: Startup fails fast in production if `JWT_SECRET`
  is unset or uses default placeholders in `authMiddleware.ts`. Public
  registration can be disabled via `ALLOW_REGISTRATION` env var and queried via
  `GET /api/v1/auth/config`. Scrub git history (`git-filter-repo` or squashed
  initial commit) prior to release to purge leaked Gemini API keys, Jellyfin
  tokens, and private domain references.
- **Test Mocking**: Unit tests executing generator or error notification
  workflows must mock `NotificationManager.prototype.notify` to avoid
  dispatching live Slack webhook requests when `SLACK_WEBHOOK_URL` is
  configured.
