---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-03T00:32:17.686223+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- Consolidated 6 microservices (PostgreSQL, Redis, Celery worker/beat, FastAPI,
  scheduler, Nginx frontend) into a single Node.js/Express container
  (`node:26-alpine` with FFmpeg) serving the React SPA, REST APIs, and media
  streams.
- Replaced PostgreSQL and Redis with filesystem JSON storage (`FileStore` in
  `data/users.json`, `data/podcasts/`, and `data/outputs/`), an in-process
  concurrency-limited `JobQueue`, and an in-process `PodcastScheduler`.
- Added migration script `migrateDbToFiles.ts` (`--db <url>`,
  `--outputs <path>`) to export legacy database records and sync on-disk episode
  manifests.

## Media Generation Pipeline
- Generates episodes via Gemini Text, Gemini Image (1:1 cover art), Gemini TTS
  multi-speaker synthesis, and FFmpeg audio assembly with ID3v2 tagging and
  Jellyfin metadata sync.
- Resolves episode disc numbers from generated titles via
  `TITLE_TO_DISC_MAPPING` in `backend/src/services/generator.ts`.
- Supports `EXTERNAL_OUTPUTS_DIR` to copy finished MP3s and append `<track>`
  metadata to `album.nfo` in external media libraries.

## Resilience & Deduplication
- Audio generation resilience: `PodcastGenerator.generateAudio` tolerates up to
  4 failed audio chunks before aborting (aborts when `failedParts >= 5`),
  allowing episodes to publish despite transient chunk errors.
- Retry handling: `GeminiService.generateAudioPart` retries up to 3 times with
  cancellable backoff on non-200 responses or API filtering (e.g., copyright
  blocks with `finishReason: 'OTHER'`), returning false instead of throwing.
- Startup deduplication: `EpisodeStore.deduplicate()` in `backend/src/index.ts`
  prunes duplicate JSON records on disk and in memory grouped by audio path or
  title (preferring records with `prompt_used` and longer scripts).
- Performance: Removed per-request `autoDiscoverEpisodes` from
  `GET /api/v1/podcasts/` to eliminate latency bottlenecks.

## Configuration & UI
- AI model configuration is driven by `TEXT_MODEL`, `IMAGE_MODEL`, and
  `AUDIO_MODEL` environment variables rather than user profile settings.
- Standardized notifications on Slack webhooks via `SLACK_WEBHOOK` with
  username hardcoded to `podcast-generator`; removed legacy Ntfy and ComfyUI
  input mounts.
- Frontend Settings page is restricted to user account and security management,
  removing user-level model and notification preferences.
