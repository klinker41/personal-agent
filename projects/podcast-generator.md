---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-08T00:34:06.244427+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- Single-Container Architecture: Consolidated 6 legacy microservices
  (PostgreSQL, Redis, Celery, FastAPI, Nginx) into an `oven/bun:alpine`
  container (Bun, Hono, FFmpeg) serving the React SPA, REST APIs, and media
  streams. Also consolidated movie generation capabilities (`movieGemini.ts`,
  `moviePipeline.ts`, `MovieStore`, routes, shared FFmpeg utilities).
- Runtime & Dependencies: Minimal backend dependencies (`hono`, `jsonwebtoken`,
  `cron-parser@4.9.0`), using native `hono/cors`, `Bun.password`, and native
  test execution (`bun test`).
- File-Based Storage & Scheduling: Replaced databases with filesystem JSON
  storage (`data/users.json`, `data/podcasts/`, `data/outputs/`), an in-process
  concurrency-limited `JobQueue`, and `PodcastScheduler`. Removed `pg` and
  legacy migration scripts (`migrateDbToFiles.ts`).
- Storage Utilities & Security: `fileStore.ts` centralizes atomic writes
  (`writeText`, `writeJson`), folder name validation, date matching, and
  Unicode NFC normalization preserving `[\p{L}\p{N}\p{M}]` for diacritics.
  Streaming endpoints enforce `path.sep` boundaries and non-blocking
  `Bun.file(path).exists()` checks to prevent path traversal.
- Code Consolidation: Unified output file streaming and route authorization
  across backend routes; deduplicated episode fetching, date parsing, and auth
  helpers on the frontend.
- Manifest Backfill: `EpisodeStore.autoDiscoverEpisodes` backfills missing
  `episode.json` files on `GET /api/podcasts/:id` (omitted from
  `GET /api/v1/podcasts/` to eliminate listing latency bottlenecks).

## Media Pipeline & Generation
- Media Generation: Synthesizes episodes via Gemini (script, 1:1 cover art,
  multi-speaker TTS) and FFmpeg audio assembly with ID3v2 tagging.
- Library Integration: Resolves disc numbers via `TITLE_TO_DISC_MAPPING` in
  `backend/src/
<truncated 617 bytes>
 (`gemini.ts`) and standardized queue error handling via `failJob` in
  `queue.ts`.
- Audio Resilience: `GeminiService.generateAudioPart` retries up to 3 times with
  cancellable backoff on non-200 responses or API filtering (e.g., copyright
  blocks with `finishReason: 'OTHER'`), returning false instead of throwing.
  `PodcastGenerator.generateAudio` aborts only after 5 failed audio chunks
  (`failedParts >= 5`).
- Deduplication & Title Matching: `EpisodeStore.deduplicate()` in
  `backend/src/index.ts` prunes disk and memory duplicates by audio path or
  title, prioritizing entries with `prompt_used` and longer scripts.
  Pre-compiles multi-strategy regexes (exact, Unicode NFC, legacy stripped
  ASCII) via `buildTitlePatterns`, guarding against empty regex patterns from
  unsafe characters.

## Configuration, Security & Testing
- Configuration & Alerts: AI models (`TEXT_MODEL`, `IMAGE_MODEL`, `AUDIO_MODEL`)
  and Slack alerts (`SLACK_WEBHOOK` or `SLACK_WEBHOOK_URL`, default username
  `podcast-generator`) configure via env vars; removed legacy Ntfy and ComfyUI.
  Episode completion alerts include canonical RSS feed URLs resolved from
  `BASE_URL` or fallback localhost port.
- UI Scope: Settings page is restricted to user account and security
  management, excluding model and alert options.
- Auth & Registration Hardening: Fails fast on startup in production if
  `JWT_SECRET` is unset or equals default placeholder values in
  `backend/src/middleware/authMiddleware.ts`. Supports `ALLOW_REGISTRATION` env
  var and `GET /api/v1/auth/config` to disable public registration and toggle UI
  links.
- Security Hygiene: Scrub git history (`git-filter-repo` or squashed initial
  commit) prior to public release to purge leaked Gemini API keys, Jellyfin
  tokens, and private domain references.
- Test Mocking: Unit tests executing generator or error notification workflows
  must mock `NotificationManager.prototype.notify` to avoid dispatching live
  Slack webhook requests when `SLACK_WEBHOOK_URL` is present in the environment.
