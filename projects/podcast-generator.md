---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-10-02T00:34:38.559284+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- **Runtime & Stack**: Containerized on `oven/bun:alpine` with Bun, Hono, and
  FFmpeg. Serves React SPA, REST APIs, streaming, and movie generation
  (`movieGemini.ts`, `moviePipeline.ts`, `MovieStore`). Minimal dependencies:
  `hono`, `jsonwebtoken`, `cron-parser@4.9.0`, native `hono/cors`,
  `Bun.password`, and `bun test`.
- **Tenancy & Persistence**: Per-user tenancy enforced via filesystem JSON
  manifests (`data/users.json`, `data/podcasts/`, `data/outputs/`) queried via
  `PodcastStore.list(user.id)` on `GET /api/podcasts` (episodes inherit
  parent tenancy), managed by in-process `JobQueue` and `PodcastScheduler`.
  Atomic writes (`writeText`, `writeJson`), path checks, and date matching
  handled by `fileStore.ts`. `EpisodeStore.autoDiscoverEpisodes` backfills
  missing `episode.json` on `GET /api/podcasts/:id` (omitted from list
  endpoints to prevent latency).

## Media Pipeline & Integrations
- **Generation & Ad Insertion**: Generates scripts, 1:1 cover art, and
  multi-speaker TTS via Gemini (`TEXT_MODEL`, `IMAGE_MODEL`, `AUDIO_MODEL`)
  tagged with FFmpeg ID3v2 metadata. Optional ad reads
  (`ad_reads?: string[] | null` from `PodcastForm.tsx` / `podcasts.ts`) are
  spaced evenly across dialogue at `k / (N + 1)` intervals via
  `formatAdReads()` in `generator.ts`.
- **Deduplication & Title Matching**: `EpisodeStore.deduplicate()`
  (`backend/src/index.ts`) prunes duplicate audio paths or titles, favoring
  records with `prompt_used` and longer scripts. `buildTitlePatterns` compiles
  exact, Unicode NFC (`[\p{L}\p{N}\p{M}]` for diacritics), and stripped ASCII
  regex patterns, guarding against empty strings.
- **Library Sync & Metadata**: Copies completed MP3s to `EXTERNAL_OUTPUTS_DIR`,
  maps disc numbers using `TITLE_TO_DISC_MAPPING`, appends `<track>` entries to
  `album.nfo`, and updates Jellyfin metadata (`POST /Items/{itemId}`) using an
  admin user ID resolved from `GET /Users`.

## Security, Resilience & Testing
- **Security & Access Control**: Startup halts in production if `JWT_SECRET`
  is unset or default (`authMiddleware.ts`). `ALLOW_REGISTRATION` toggles user
  signups (`GET /api/v1/auth/config`); settings UI is restricted to
  account/security. Streaming verifies `path.sep` boundaries and non-blocking
  `Bun.file(path).exists()` against path traversal. Git history must be purged
  (`git-filter-repo` or squashed) before release to remove leaked Gemini keys,
  Jellyfin tokens, and private domains.
- **Resilience & Error Handling**: `postWithRetry` (`gemini.ts`) provides
  exponential backoff; queue errors standardize on `failJob` (`queue.ts`).
  `GeminiService.generateAudioPart` retries up to 3 times on non-200 responses
  or API filtering (returns false without throwing), aborting if
  `failedParts >= 5`.
- **Notifications & Test Mocking**: Slack alerts (`SLACK_WEBHOOK` or
  `SLACK_WEBHOOK_URL`, sender `podcast-generator`) embed canonical RSS feed
  URLs via `BASE_URL` or fallback port. Generator and notification unit tests
  must mock `NotificationManager.prototype.notify` to prevent live webhooks.
