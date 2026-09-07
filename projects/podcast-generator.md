---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-07T00:34:20.222610+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- Single-Container Architecture: Consolidated 6 legacy microservices (Postgres,
  Redis, Celery, FastAPI, Nginx) into an `oven/bun:alpine` container (Bun, Hono,
  FFmpeg) serving the React SPA, REST APIs, and media streams. Also consolidated
  movie generation capabilities (`movieGemini.ts`, `moviePipeline.ts`,
  `MovieStore`, routes, shared ffmpeg utilities).
- Runtime & Dependencies: Reduced backend runtime dependencies to three packages
  (`hono`, `jsonwebtoken`, `cron-parser@4.9.0`), using native `hono/cors`,
  `Bun.password`, and native test execution (`bun test`).
- File-Based Storage: Replaced databases with filesystem JSON storage
  (`data/users.json`, `data/podcasts/`, `data/outputs/`), an in-process
  concurrency-limited `JobQueue`, and `PodcastScheduler`. Removed `pg` and
  legacy DB migration scripts (previously exported via `migrateDbToFiles.ts`).
- Storage Utilities & Security: `fileStore.ts` centralizes atomic write logic
  (`writeText`, `writeJson`), folder name validation, and date matching. Audio
  and cover streaming endpoints enforce `path.sep` boundaries and non-blocking
  `Bun.file(path).exists()` checks to prevent path traversal.
- Code Consolidation: Unified output file streaming and route authorization
  across backend routes; deduplicated episode fetching, date parsing, and auth
  helpers on the frontend.
- Manifest Backfill: `EpisodeStore.autoDiscoverEpisodes` backfills missing
  `episode.json` files on `GET /api/podcasts/:id` (removed from
  `GET /api/v1/podcasts/` to eliminate listing latency bottlenecks).

## Media Pipeline & Library Integration
- Media Generation: Generates episodes via Gemini (script, 1:1 cover art,
  multi-speaker TTS) and FFmpeg audio assembly with ID3v2 tagging.
- Library Integration: Resolves disc numbers via `TITLE_TO_DISC_MAPPING` in
  `backend/src/services/
<truncated 1900 bytes>
erateAudioPart` retries up to 3 times with
  cancellable backoff on non-200 responses or API filtering (e.g., copyright
  blocks with `finishReason: 'OTHER'`), returning false instead of throwing.
  `PodcastGenerator.generateAudio` aborts only after 5 failed audio chunks
  (`failedParts >= 5`).
- Deduplication & Title Matching: `EpisodeStore.deduplicate()` in
  `backend/src/index.ts` prunes disk and memory duplicates by audio path or
  title, prioritizing entries with `prompt_used` and longer scripts.
  Pre-compiles multi-strategy regexes (exact, Unicode NFC, legacy stripped
  ASCII) via `buildTitlePatterns` (guarding against empty patterns from unsafe
  characters). `fileStore.ts` normalizes Unicode NFC and preserves
  `[\p{L}\p{N}\p{M}]` to retain diacritics and non-ASCII characters.

## Configuration, Security & Testing
- Configuration & Alerts: AI models (`TEXT_MODEL`, `IMAGE_MODEL`, `AUDIO_MODEL`)
  and Slack alerts (`SLACK_WEBHOOK` or `SLACK_WEBHOOK_URL`, default username
  `podcast-generator`) configure via env vars; removed legacy Ntfy and ComfyUI.
  Episode completion Slack alerts include canonical RSS feed URLs resolved from
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
