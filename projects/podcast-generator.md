---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-10-13T00:36:34.568202+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- **Stack & Runtime**: Runs on `oven/bun:alpine` with Bun, Hono, and FFmpeg;
  serves React SPA, REST APIs, audio streaming, and movie pipelines
  (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`). Minimal external
  dependencies include `jsonwebtoken` and `cron-parser@4.9.0` alongside
  built-in `hono/cors`, `Bun.password`, and `bun test`.
- **Tenancy & Storage**: Manifest-backed user isolation (`data/users.json`,
  `data/podcasts/`, `data/outputs/`) with episode tenancy queried via
  `PodcastStore.list(user.id)` on `GET /api/podcasts`. `fileStore.ts`
  provides path-safe atomic writes (`writeText`, `writeJson`) and date
  matching.
- **Background Tasks & Discovery**: In-process `JobQueue` and
  `PodcastScheduler` manage tasks. `EpisodeStore.autoDiscoverEpisodes`
  backfills missing `episode.json` on `GET /api/podcasts/:id` (omitted from
  listings to prevent latency).

## Media Pipeline & Integrations
- **Generation & Ads**: Gemini (`TEXT_MODEL`, `IMAGE_MODEL`, `AUDIO_MODEL`)
  generates scripts, 1:1 cover art, and multi-speaker TTS tagged with FFmpeg
  ID3v2. `formatAdReads()` (`generator.ts`) spaces ad reads (`ad_reads?:
  string[] | null` via `PodcastForm.tsx` / `podcasts.ts`) evenly at
  `k / (N + 1)` intervals.
- **Deduplication & Matching**: `EpisodeStore.deduplicate()`
  (`backend/src/index.ts`) drops duplicate paths/titles, favoring entries with
  `prompt_used` and longer scripts. `buildTitlePatterns` compiles exact, Unicode
  NFC (`[\p{L}\p{N}\p{M}]`), and stripped ASCII regexes, guarding against empty
  strings.
- **Jellyfin Sync**: Copies MP3s to `EXTERNAL_OUTPUTS_DIR`, maps disc numbers
  via `TITLE_TO_DISC_MAPPING`, appends `<track>` entries to `album.nfo`, and
  triggers Jellyfin sync (`POST /Items/{itemId}`) using an admin user resolved
  from `GET /Users`.

## Security, Resilience & Testing
- **Auth & Security**: Startup halts if `JWT_SECRET` is unset or default
  (`authMiddleware.ts`). `ALLOW_REGISTRATION` toggles signups
  (`GET /api/v1/auth/config`); settings UI is restricted to account/security.
  Streaming validates `path.sep` boundaries via non-blocking
  `Bun.file(path).exists()`. Purge git history (`git-filter-repo` or squash)
  prior to release to remove leaked secrets and internal domains.
- **Resilience & Retries**: `postWithRetry` (`gemini.ts`) applies exponential
  backoff; queue failures standardize on `failJob` (`queue.ts`).
  `GeminiService.generateAudioPart` retries up to 3 times on non-200 or content
  filtering (returns false without throwing), aborting if `failedParts >= 5`.
- **Alerts & Test Isolation**: Slack alerts (`SLACK_WEBHOOK` or
  `SLACK_WEBHOOK_URL`, sender `podcast-generator`) include canonical RSS feed
  URLs via `BASE_URL` or fallback port. Tests for generation and notifications
  must mock `NotificationManager.prototype.notify` to prevent live webhook
  dispatches.
