---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-26T00:34:26.464571+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- **Runtime & Stack**: Single `oven/bun:alpine` container running Bun, Hono,
  and FFmpeg for React SPA, REST APIs, streaming, and movie generation
  (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`). Minimal dependencies:
  `hono`, `jsonwebtoken`, `cron-parser@4.9.0`, native `hono/cors`,
  `Bun.password`, and `bun test`.
- **Persistence & Tenancy**: Filesystem JSON manifests (`data/users.json`,
  `data/podcasts/`, `data/outputs/`) enforce per-user tenancy
  (`PodcastStore.list(user.id)` on `GET /api/podcasts`; episodes inherit
  tenancy), managed by in-process `JobQueue` and `PodcastScheduler`.
  `fileStore.ts` provides atomic writes (`writeText`, `writeJson`), directory
  validation, and date matching. `EpisodeStore.autoDiscoverEpisodes` backfills
  missing `episode.json` on `GET /api/podcasts/:id` (omitted from list
  endpoints to avoid latency).

## Media Pipeline & Integrations
- **Synthesis & Ad Placement**: Synthesizes scripts, 1:1 cover art, and
  multi-speaker TTS via Gemini (`TEXT_MODEL`, `IMAGE_MODEL`, `AUDIO_MODEL`)
  with FFmpeg ID3v2 tagging. Optional ad reads (`ad_reads?: string[] | null`
  from `PodcastForm.tsx` / `podcasts.ts`) are spaced evenly across dialogue at
  intervals `k / (N + 1)` via `formatAdReads()` in `generator.ts`.
- **Deduplication & Matching**: `EpisodeStore.deduplicate()` in
  `backend/src/index.ts` prunes duplicates by audio path or title, favoring
  entries with `prompt_used` and longer scripts. `buildTitlePatterns` compiles
  regexes (exact, Unicode NFC preserving `[\p{L}\p{N}\p{M}]` for diacritics,
  stripped ASCII), guarding against empty patterns from invalid characters.
- **Library Sync & Metadata**: Copies finished MP3s to `EXTERNAL_OUTPUTS_DIR`,
  maps disc numbers via `TITLE_TO_DISC_MAPPING`, appends `<track>` entries to
  `album.nfo`, and updates Jellyfin metadata (`POST /Items/{itemId}`) using an
  admin user ID resolved from `GET /Users`.

## Resilience, Security & Testing
- **Auth Hardening & Traversal Defense**: Startup fails fast in production if
  `JWT_SECRET` is unset or default in `authMiddleware.ts`. Registration is
  toggleable via `ALLOW_REGISTRATION` (`GET /api/v1/auth/config`); settings
  UI is limited to account/security. Media streaming prevents path traversal
  via `path.sep` boundaries and non-blocking `Bun.file(path).exists()`. Purge
  Git history (`git-filter-repo` or squashed commit) before release to remove
  leaked secrets (Gemini keys, Jellyfin tokens, private domains).
- **Error Handling & Retries**: `postWithRetry` (`gemini.ts`) provides
  centralized exponential backoff; queue errors standardize on `failJob`
  (`queue.ts`). `GeminiService.generateAudioPart` retries up to 3 times on
  non-200 responses or API filtering (returns false without throwing).
  `PodcastGenerator.generateAudio` aborts when `failedParts >= 5`.
- **Alerts & Test Requirements**: Sends Slack alerts via `SLACK_WEBHOOK` or
  `SLACK_WEBHOOK_URL` (default user `podcast-generator`). Completion alerts
  include canonical RSS feed URLs from `BASE_URL` or fallback port. Generator
  and notification unit tests must mock `NotificationManager.prototype.notify`
  to avoid live Slack webhook calls.
