---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-10-05T00:35:49.914689+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- **Runtime & Stack**: Containerized on `oven/bun:alpine` with Bun, Hono, and
  FFmpeg. Serves React SPA, REST APIs, audio streaming, and movie pipelines
  (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`). Minimal dependencies:
  `hono`, `jsonwebtoken`, `cron-parser@4.9.0`, plus built-in `hono/cors`,
  `Bun.password`, and `bun test`.
- **Tenancy & State**: Per-user filesystem manifests (`data/users.json`,
  `data/podcasts/`, `data/outputs/`) queried via `PodcastStore.list(user.id)`
  on `GET /api/podcasts` (episodes inherit tenancy). Atomic writes
  (`writeText`, `writeJson`), path checks, and date matching are handled in
  `fileStore.ts`. Background tasks run via in-process `JobQueue` and
  `PodcastScheduler`.
- **Episode Discovery**: `EpisodeStore.autoDiscoverEpisodes` backfills missing
  `episode.json` on `GET /api/podcasts/:id` (omitted from list queries to avoid
  latency).

## Media Pipeline & Integrations
- **Generation & Ads**: Produces scripts, 1:1 cover art, and multi-speaker TTS
  using Gemini (`TEXT_MODEL`, `IMAGE_MODEL`, `AUDIO_MODEL`) with FFmpeg ID3v2
  tags. Spaced ad reads (`ad_reads?: string[] | null` from `PodcastForm.tsx` /
  `podcasts.ts`) are inserted evenly at `k / (N + 1)` intervals via
  `formatAdReads()` in `generator.ts`.
- **Deduplication & Matching**: `EpisodeStore.deduplicate()`
  (`backend/src/index.ts`) drops duplicate paths/titles, favoring records with
  `prompt_used` and longer scripts. `buildTitlePatterns` compiles exact,
  Unicode NFC (`[\p{L}\p{N}\p{M}]`), and stripped ASCII regexes, guarding
  against empty strings.
- **Jellyfin & Library Sync**: Copies MP3s to `EXTERNAL_OUTPUTS_DIR`, maps disc
  numbers via `TITLE_TO_DISC_MAPPING`, appends `<track>` entries to `album.nfo`,
  and updates Jellyfin (`POST /Items/{itemId}`) using an admin user resolved
  via `GET /Users`.

## Security, Resilience & Testing
- **Auth & Traversal**: Startup halts if `JWT_SECRET` is unset or default
  (`authMiddleware.ts`). `ALLOW_REGISTRATION` toggles signups
  (`GET /api/v1/auth/config`); settings UI is scoped to account and security.
  Streaming validates `path.sep` boundaries via non-blocking
  `Bun.file(path).exists()`. Purge git history (`git-filter-repo` or squash)
  prior to release to strip leaked keys, tokens, and private domains.
- **Retry Mechanics**: `postWithRetry` (`gemini.ts`) applies exponential
  backoff; queue failures standardize on `failJob` (`queue.ts`). In
  `GeminiService`, `generateAudioPart` retries up to 3 times on non-200 or
  content filtering (returns false without throwing), aborting if
  `failedParts >= 5`.
- **Alerts & Test Isolation**: Slack alerts (`SLACK_WEBHOOK` or
  `SLACK_WEBHOOK_URL`, sender `podcast-generator`) include canonical RSS feed
  URLs via `BASE_URL` or fallback port. Generation and notification tests must
  mock `NotificationManager.prototype.notify` to prevent live webhook
  dispatches.
