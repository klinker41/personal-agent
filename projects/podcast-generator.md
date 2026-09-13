---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-13T00:32:40.900368+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- Single-Container Runtime: Consolidated 6 legacy microservices (PostgreSQL,
  Redis, Celery, FastAPI, Nginx) into an `oven/bun:alpine` container running
  Bun, Hono, and FFmpeg for the React SPA, REST APIs, media streaming, and
  movie generation (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`). Uses
  minimal dependencies (`hono`, `jsonwebtoken`, `cron-parser@4.9.0`), native
  `hono/cors`, `Bun.password`, and `bun test`.
- Persistence & Tenancy: Replaced database dependencies (`pg`, migrations) with
  filesystem JSON storage (`data/users.json`, `data/podcasts/`,
  `data/outputs/`), an in-process concurrency-limited `JobQueue`, and
  `PodcastScheduler`. Manifests enforce per-user tenancy via `user_id`
  (`GET /api/podcasts` filters via `PodcastStore.list(user.id)`; episodes
  inherit tenancy). Manifests support `ad_reads?: string[] | null` edited in
  `PodcastForm.tsx` and handled in `podcasts.ts`.
- Manifest Discovery: `EpisodeStore.autoDiscoverEpisodes` backfills missing
  `episode.json` on `GET /api/podcasts/:id` but is omitted from listing endpoint
  `GET /api/v1/podcasts/` to eliminate latency bottlenecks.
- File Utilities & Security: `fileStore.ts` provides atomic writes (`writeText`,
  `writeJson`), folder validation, date matching, and Unicode NFC normalization
  preserving `[\p{L}\p{N}\p{M}]` for diacritics. Audio and cover streaming
  enforces `path.sep` boundaries and non-blocking `Bun.file(path).exists()`
  checks against path traversal. Unified streaming and route auth helpers.

## Media Pipeline, Integrations & Resilience
- Generation & Ad Placement: Synthesizes episodes via Gemini (script, 1:1 cover
  art, multi-speaker TTS) and FFmpeg ID3v2 tagging. Distributes N ad reads
  evenly at fractional intervals `k / (N + 1)` across dialogue via
  `formatAdReads()` in `generator.ts`.
- External Libr
<truncated 178 bytes>
OUTPUTS_DIR`, and syncs
  Jellyfin item metadata by resolving an admin user ID to supply as `userId` on
  `GET /Items/{itemId}` before updating via `POST /Items/{itemId}`.
- Error Handling & Resilience: HTTP calls use exponential backoff via
  `postWithRetry` (`gemini.ts`), and queue errors standardize on `failJob` in
  `queue.ts`. `GeminiService.generateAudioPart` retries up to 3 times on
  non-200 responses or API filtering (returning false rather than throwing).
  `PodcastGenerator.generateAudio` aborts only after 5 failed chunks
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
  `podcast-generator`) configure via environment variables (removed legacy Ntfy
  and ComfyUI). Completion alerts include canonical RSS feed URLs from
  `BASE_URL` or fallback port. Settings UI is restricted to account/security.
- Auth Hardening & Hygiene: Startup fails fast in production if `JWT_SECRET` is
  unset or uses default placeholders in
  `backend/src/middleware/authMiddleware.ts`. Public registration can be
  disabled via `ALLOW_REGISTRATION` env var and queried via
  `GET /api/v1/auth/config`. Scrub git history (`git-filter-repo` or squashed
  initial commit) prior to release to purge leaked Gemini API keys, Jellyfin
  tokens, and private domain references.
- Test Mocking: Unit tests executing generator or error notification workflows
  must mock `NotificationManager.prototype.notify` to avoid dispatching live
  Slack webhook requests when `SLACK_WEBHOOK_URL` is configured.
