---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-06T00:32:27.975397+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- Single-Container Architecture: Consolidated 6 legacy microservices (Postgres,
  Redis, Celery, FastAPI, Nginx) into an `oven/bun:alpine` container (Bun, Hono,
  FFmpeg) serving the React SPA, REST APIs, and media streams.
- Runtime & Dependencies: Reduced backend runtime dependencies to three packages
  (`hono`, `jsonwebtoken`, `cron-parser@4.9.0`), using native `hono/cors`,
  `Bun.password`, and native test execution (`bun test`).
- File-Based Storage: Replaced databases with filesystem JSON storage
  (`data/users.json`, `data/podcasts/`, `data/outputs/`), an in-process
  concurrency-limited `JobQueue`, and `PodcastScheduler`. Removed `pg` and
  legacy DB migration scripts (previously exported via `migrateDbToFiles.ts`).
- Storage Utilities: `fileStore.ts` centralizes atomic write logic (`writeText`,
  `writeJson`), folder name validation, and date matching patterns.
- Streaming Security: Audio and cover streaming endpoints validate `path.sep`
  boundaries and perform non-blocking `Bun.file(path).exists()` checks to
  prevent path traversal.
- Code Consolidation: Unified output file streaming and route authorization
  across backend routes; deduplicated episode fetching, date parsing, and auth
  helpers on the frontend.

## Media Pipeline & Library Integration
- Media Generation: Generates episodes via Gemini (script, 1:1 cover art,
  multi-speaker TTS) and FFmpeg audio assembly with ID3v2 tagging.
- Library Integration: Resolves disc numbers via `TITLE_TO_DISC_MAPPING` in
  `backend/src/services/generator.ts`; copies finished MP3s and appends
  `<track>` metadata to `album.nfo` via `EXTERNAL_OUTPUTS_DIR`.

## Prompt Architecture & Speaker Profiles
- Preamble & Target Length: Automatically prepends podcast title and word count
  bounds to base prompts using `target_length` presets: XS (500-1500), S
  (1000-3000), M 
<truncated 1869 bytes>
gacy stripped ASCII). Pre-compiles regexes via
  `buildTitlePatterns` and guards against empty patterns from unsafe characters.
- Title Sanitization: `fileStore.ts` normalizes to Unicode NFC and preserves
  `[\p{L}\p{N}\p{M}]` to retain diacritics and non-ASCII characters.
- Manifest Backfill: `EpisodeStore.autoDiscoverEpisodes` backfills missing
  `episode.json` files on `GET /api/podcasts/:id` (removed from
  `GET /api/v1/podcasts/` to eliminate listing latency bottlenecks).

## Resilience & Deduplication
- Retries & Backoff: Centralized HTTP exponential backoff in `postWithRetry`
  (`gemini.ts`) and standardized queue error handling via `failJob` in
  `queue.ts`.
- Audio Resilience: `GeminiService.generateAudioPart` retries up to 3 times with
  cancellable backoff on non-200 responses or API filtering (e.g., copyright
  blocks with `finishReason: 'OTHER'`), returning false instead of throwing.
  `PodcastGenerator.generateAudio` aborts only after 5 failed audio chunks
  (`failedParts >= 5`).
- Deduplication: `EpisodeStore.deduplicate()` in `backend/src/index.ts` prunes
  disk and memory duplicates by audio path or title, prioritizing entries with
  `prompt_used` and longer scripts.

## Configuration & Security
- Configuration: AI models (`TEXT_MODEL`, `IMAGE_MODEL`, `AUDIO_MODEL`) and
  alerts (`SLACK_WEBHOOK` or `SLACK_WEBHOOK_URL`, default username
  `podcast-generator`) configure via env vars. Removed legacy Ntfy and ComfyUI.
- UI Scope: Settings page is restricted to user account and security management,
  excluding model and alert options.
- Security Requirements:
  - Validate `JWT_SECRET` in `backend/src/middleware/authMiddleware.ts` on
    startup and fail fast in production.
  - Implement registration controls in `backend/src/routes/auth.ts` (e.g.,
    invite codes or disable registration post-admin) to protect Gemini quota.
  - Scrub git history (`git-filter-repo` or squashed initial commit) prior to
    public release to purge leaked Gemini API keys, Jellyfin tokens, and
    private domain references.
