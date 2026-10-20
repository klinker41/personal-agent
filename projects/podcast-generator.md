---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-10-20T06:39:25.184165+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- **Stack & Runtime**: Built on `oven/bun:alpine` with Bun, Hono, and FFmpeg,
  serving a React SPA, REST APIs, audio streaming, and movie pipelines
  (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`). Dependencies are kept
  minimal (`jsonwebtoken`, `cron-parser@4.9.0`), using `hono/cors`,
  `Bun.password`, and `bun test`.
- **Tenancy & Persistence**: Isolated via `data/users.json`, `data/podcasts/`,
  and `data/outputs/` (`GET /api/podcasts` resolves tenancy via
  `PodcastStore.list(user.id)`). `fileStore.ts` provides atomic writes
  (`writeText`, `writeJson`) and date matching.
- **Scheduling & Discovery**: Tasks run via in-process `JobQueue` and
  `PodcastScheduler`. `EpisodeStore.autoDiscoverEpisodes` lazily backfills
  missing `episode.json` on `GET /api/podcasts/:id` (omitted from listings to
  prevent latency).

## Media Pipeline & Integrations
- **Generation & Ad Insertion**: Gemini (`TEXT_MODEL`, `IMAGE_MODEL`,
  `AUDIO_MODEL`) produces scripts, 1:1 cover art, and multi-speaker TTS with
  FFmpeg ID3v2 tags. `formatAdReads()` (`generator.ts`) evenly spaces ad reads
  (`ad_reads?: string[] | null` via `PodcastForm.tsx` / `podcasts.ts`) at
  `k / (N + 1)` intervals.
- **Deduplication & Matching**: `EpisodeStore.deduplicate()`
  (`backend/src/index.ts`) drops duplicate paths and titles, favoring longer
  scripts and records with `prompt_used`. `buildTitlePatterns` compiles exact,
  Unicode NFC (`[\p{L}\p{N}\p{M}]`), and stripped ASCII regexes, guarding
  against empty strings.
- **Jellyfin Sync**: Copies MP3s to `EXTERNAL_OUTPUTS_DIR`, maps disc numbers
  via `TITLE_TO_DISC_MAPPING`, appends `<track>` entries to `album.nfo`, and
  triggers Jellyfin sync (`POST /Items/{itemId}`) using an admin discovered
  from `GET /Users`.

## Security, Reliability & Testing
- **Auth & Access Control**: Startup halts if `JWT_SECRET` is unset or default
  (`authMiddleware.ts`). `ALLOW_REGISTRATION` governs signups
  (`GET /api/v1/auth/config`); settings UI is restricted to account/security.
  Streaming validates `path.sep` boundaries via non-blocking
  `Bun.file(path).exists()`. Scrub git history (`git-filter-repo` or squash)
  before release to remove secrets and internal domains.
- **Retries & Error Handling**: `postWithRetry` (`gemini.ts`) applies
  exponential backoff; queue failures invoke `failJob` (`queue.ts`).
  `GeminiService.generateAudioPart` retries up to 3 times on non-200 responses
  or content filtering (returns false without throwing), aborting if
  `failedParts >= 5`.
- **Alerts & Test Isolation**: Slack alerts (`SLACK_WEBHOOK` or
  `SLACK_WEBHOOK_URL`, sender `podcast-generator`) build canonical RSS feed
  URLs with `BASE_URL` or fallback port. Notification and generation tests
  must mock `NotificationManager.prototype.notify` to prevent live dispatches.
