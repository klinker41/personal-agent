---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-10-04T00:38:19.176691+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- **Runtime & Stack**: Containerized on `oven/bun:alpine` with Bun, Hono, and
  FFmpeg. Serves React SPA, REST APIs, audio streaming, and movie generation
  (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`). Minimal external
  dependencies: `hono`, `jsonwebtoken`, and `cron-parser@4.9.0`, alongside
  built-in `hono/cors`, `Bun.password`, and `bun test`.
- **Tenancy & Persistence**: Per-user tenancy via filesystem JSON manifests
  (`data/users.json`, `data/podcasts/`, `data/outputs/`) queried through
  `PodcastStore.list(user.id)` on `GET /api/podcasts` (episodes inherit
  parent tenancy). Background tasks use in-process `JobQueue` and
  `PodcastScheduler`. `fileStore.ts` provides atomic writes (`writeText`,
  `writeJson`), path boundary checks, and date matching.
- **Episode Discovery**: `EpisodeStore.autoDiscoverEpisodes` backfills missing
  `episode.json` on `GET /api/podcasts/:id` (omitted from list endpoints to
  avoid latency).

## Media Pipeline & Integrations
- **Generation & Ad Insertion**: Generates scripts, 1:1 cover art, and
  multi-speaker TTS using Gemini (`TEXT_MODEL`, `IMAGE_MODEL`, and
  `AUDIO_MODEL`), tagged with FFmpeg ID3v2 metadata. Spaced ad reads
  (`ad_reads?: string[] | null` from `PodcastForm.tsx` / `podcasts.ts`) are
  inserted evenly across dialogue at `k / (N + 1)` intervals via
  `formatAdReads()` in `generator.ts`.
- **Deduplication & Matching**: `EpisodeStore.deduplicate()`
  (`backend/src/index.ts`) removes duplicate audio paths or titles,
  preferring records with `prompt_used` and longer scripts.
  `buildTitlePatterns` compiles exact, Unicode NFC (`[\p{L}\p{N}\p{M}]` for
  diacritics), and stripped ASCII regexes, guarding against empty strings.
- **Library Sync & Jellyfin**: Copies completed MP3s to `EXTERNAL_OUTPUTS_DIR`,
  maps disc numbers via `TITLE_TO_DISC_MAPPING`, appends `<track>` entries to
  `album.nfo`, and syncs metadata to Jellyfin (`POST /Items/{itemId}`) using
  an admin user ID resolved via `GET /Users`.

## Security, Resilience & Testing
- **Security & Access Control**: Production startup halts if `JWT_SECRET` is
  unset or default (`authMiddleware.ts`). `ALLOW_REGISTRATION` toggles user
  signups (`GET /api/v1/auth/config`); settings UI is restricted to account
  and security. Streaming enforces `path.sep` boundaries and non-blocking
  `Bun.file(path).exists()` traversal checks. Git history must be purged
  (`git-filter-repo` or squash) before release to strip leaked Gemini keys,
  Jellyfin tokens, and private domains.
- **Resilience & Retries**: `postWithRetry` (`gemini.ts`) provides exponential
  backoff; queue errors standardize on `failJob` (`queue.ts`).
  `GeminiService.generateAudioPart` retries up to 3 times on non-200 responses
  or API content filtering (returns false without throwing), aborting if
  `failedParts >= 5`.
- **Slack Alerts & Test Mocking**: Slack alerts (`SLACK_WEBHOOK` or
  `SLACK_WEBHOOK_URL`, sender `podcast-generator`) embed canonical RSS feed
  URLs via `BASE_URL` or fallback port. Unit tests for generation and
  notifications must mock `NotificationManager.prototype.notify` to prevent
  live webhook calls.
