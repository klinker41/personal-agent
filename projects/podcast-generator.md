---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-10-16T00:38:19.883844+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- **Stack & Runtime**: Runs on `oven/bun:alpine` with Bun, Hono, and FFmpeg,
  serving a React SPA, REST APIs, audio streaming, and movie pipelines
  (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`). Minimal external
  dependencies include `jsonwebtoken` and `cron-parser@4.9.0` alongside
  built-in `hono/cors`, `Bun.password`, and `bun test`.
- **Tenancy & Storage**: Manifest-backed user isolation (`data/users.json`,
  `data/podcasts/`, `data/outputs/`) queries episode tenancy via
  `PodcastStore.list(user.id)` on `GET /api/podcasts`. `fileStore.ts`
  provides date matching and path-safe atomic writes (`writeText`,
  `writeJson`).
- **Task Scheduling & Discovery**: In-process `JobQueue` and
  `PodcastScheduler` manage tasks. `EpisodeStore.autoDiscoverEpisodes`
  backfills missing `episode.json` on `GET /api/podcasts/:id` (omitted from
  listings to prevent latency).

## Media Pipeline & Integrations
- **Generation & Ad Insertion**: Gemini (`TEXT_MODEL`, `IMAGE_MODEL`,
  `AUDIO_MODEL`) produces scripts, 1:1 cover art, and multi-speaker TTS tagged
  via FFmpeg ID3v2. `formatAdReads()` (`generator.ts`) evenly spaces ad reads
  (`ad_reads?: string[] | null` via `PodcastForm.tsx` / `podcasts.ts`) at
  `k / (N + 1)` intervals.
- **Deduplication & Title Matching**: `EpisodeStore.deduplicate()`
  (`backend/src/index.ts`) drops duplicate paths and titles, prioritizing
  longer scripts and entries with `prompt_used`. `buildTitlePatterns` compiles
  exact, Unicode NFC (`[\p{L}\p{N}\p{M}]`), and stripped ASCII regexes,
  guarding against empty strings.
- **Jellyfin Sync**: Copies MP3s to `EXTERNAL_OUTPUTS_DIR`, maps disc numbers
  via `TITLE_TO_DISC_MAPPING`, appends `<track>` entries to `album.nfo`, and
  triggers Jellyfin sync (`POST /Items/{itemId}`) via an admin user discovered
  from `GET /Users`.

## Security, Resilience & Testing
- **Auth & Access Control**: Startup halts if `JWT_SECRET` is unset or default
  (`authMiddleware.ts`). `ALLOW_REGISTRATION` controls signups
  (`GET /api/v1/auth/config`), and settings UI is limited to account/security.
  Media streaming validates `path.sep` boundaries via non-blocking
  `Bun.file(path).exists()`. Purge git history (`git-filter-repo` or squash)
  before release to scrub secrets and internal domains.
- **Error Handling & Retries**: `postWithRetry` (`gemini.ts`) applies
  exponential backoff, and queue failures standardize on `failJob`
  (`queue.ts`). `GeminiService.generateAudioPart` retries up to 3 times on
  non-200 responses or content filtering (returning false without throwing),
  aborting if `failedParts >= 5`.
- **Alerts & Test Isolation**: Slack alerts (`SLACK_WEBHOOK` or
  `SLACK_WEBHOOK_URL`, sender `podcast-generator`) format canonical RSS feed
  URLs using `BASE_URL` or fallback port. Generation and notification tests
  must mock `NotificationManager.prototype.notify` to avoid live webhook
  dispatches.
