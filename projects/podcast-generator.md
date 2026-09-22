---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-22T00:33:58.112181+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- **Single-Container Runtime**: Consolidated legacy services (Postgres, Redis,
  Celery, FastAPI, Nginx, ComfyUI, Ntfy) into an `oven/bun:alpine` container.
  Runs Bun, Hono, and FFmpeg for the React SPA, REST APIs, streaming, and movie
  generation (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`). Minimal
  dependencies: `hono`, `jsonwebtoken`, `cron-parser@4.9.0`, native
  `hono/cors`, `Bun.password`, and `bun test`.
- **Filesystem Persistence & Tenancy**: Filesystem JSON manifests
  (`data/users.json`, `data/podcasts/`, `data/outputs/`) enforce per-user
  tenancy (`PodcastStore.list(user.id)` on `GET /api/podcasts`; episodes
  inherit tenancy). Managed by in-process, concurrency-limited `JobQueue` and
  `PodcastScheduler`. `fileStore.ts` provides atomic writes (`writeText`,
  `writeJson`), directory validation, and date matching.
  `EpisodeStore.autoDiscoverEpisodes` backfills missing `episode.json` on
  `GET /api/podcasts/:id` (omitted from list endpoints to prevent latency).

## Media Pipeline & Integrations
- **Generation & Ad Placement**: Synthesizes episodes via Gemini
  (`TEXT_MODEL`, `IMAGE_MODEL`, `AUDIO_MODEL` for script, 1:1 cover art,
  multi-speaker TTS) and FFmpeg ID3v2 tagging. Distributes optional ad reads
  (`ad_reads?: string[] | null` from `PodcastForm.tsx` / `podcasts.ts`) evenly
  across dialogue at intervals `k / (N + 1)` via `formatAdReads()` in
  `generator.ts`.
- **Deduplication & Matching**: `EpisodeStore.deduplicate()` in
  `backend/src/index.ts` prunes duplicate records by audio path or title,
  preferring entries with `prompt_used` and longer scripts.
  `buildTitlePatterns` pre-compiles regexes (exact, Unicode NFC preserving
  `[\p{L}\p{N}\p{M}]` for diacritics, stripped ASCII), guarding against empty
  regexes from unsafe characters.
- **Library Sync & Metadata**: Copies finished MP3s, maps disc numbers via
  `TITLE_TO_DISC_MAPPING`, appends `<track>` metadata to `album.nfo` in
  `EXTERNAL_OUTPUTS_DIR`, and updates Jellyfin item metadata
  (`POST /Items/{itemId}`) using an admin user ID resolved from `GET /Users`.

## Resilience, Security & Testing
- **Auth Hardening & Traversal Defense**: Startup fails fast in production if
  `JWT_SECRET` is unset or default in `authMiddleware.ts`. Public registration
  is toggleable via `ALLOW_REGISTRATION` (`GET /api/v1/auth/config`); settings
  UI is limited to account/security. Media streaming prevents path traversal
  via `path.sep` boundaries and non-blocking `Bun.file(path).exists()`. Purge
  Git history (`git-filter-repo` or squashed commit) before release to remove
  leaked secrets (Gemini keys, Jellyfin tokens, private domains).
- **Error Handling & Retries**: Centralized exponential backoff handles HTTP
  calls via `postWithRetry` (`gemini.ts`); queue errors standardize on
  `failJob` (`queue.ts`). `GeminiService.generateAudioPart` retries up to 3
  times on non-200 responses or API filtering (returns false without throwing).
  `PodcastGenerator.generateAudio` aborts only when `failedParts >= 5`.
- **Alerts & Test Requirements**: Sends Slack alerts via `SLACK_WEBHOOK` or
  `SLACK_WEBHOOK_URL` (default user `podcast-generator`). Completion alerts
  include canonical RSS feed URLs from `BASE_URL` or fallback port. Generator
  and notification unit tests must mock `NotificationManager.prototype.notify`
  to avoid live Slack webhook calls.
