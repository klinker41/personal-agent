---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-05T00:32:19.059778+00:00
confidence: 0.95
---

# Project: Podcast-Generator

## Architecture & Storage
- Consolidated 6 microservices (PostgreSQL, Redis, Celery worker/beat, FastAPI,
  scheduler, Nginx) into a single `node:26-alpine` container (Node.js/Express
  and FFmpeg) serving the React SPA, REST APIs, and media streams.
- Storage & Scheduling: Replaced database dependencies with filesystem JSON
  storage (`FileStore` in `data/users.json`, `data/podcasts/`,
  `data/outputs/`), an in-process concurrency-limited `JobQueue`, and
  `PodcastScheduler`.
- Migration: `migrateDbToFiles.ts` (`--db <url>`, `--outputs <path>`) exports
  legacy database records and syncs on-disk episode manifests.

## Media Generation & Library Sync
- Pipeline generates episodes via Gemini (Text, 1:1 Image cover art,
  multi-speaker TTS) and FFmpeg audio assembly with ID3v2 tags and Jellyfin
  metadata sync.
- Library Integration: Resolves disc numbers via `TITLE_TO_DISC_MAPPING` in
  `backend/src/services/generator.ts`; copies finished MP3s and appends
  `<track>` metadata to `album.nfo` via `EXTERNAL_OUTPUTS_DIR`.

## Resilience & Deduplication
- Audio Resilience: `GeminiService.generateAudioPart` retries up to 3 times
  with cancellable backoff on non-200 responses or API filtering (e.g.,
  copyright blocks with `finishReason: 'OTHER'`), returning false instead of
  throwing. `PodcastGenerator.generateAudio` tolerates up to 4 failed audio
  chunks before aborting (`failedParts >= 5`).
- Deduplication & Performance: `EpisodeStore.deduplicate()` in
  `backend/src/index.ts` prunes disk and memory duplicates by audio path or
  title (preferring records with `prompt_used` and longer scripts). Removed
  per-request `autoDiscoverEpisodes` from `GET /api/v1/podcasts/` to eliminate
  latency bottlenecks.

## Configuration & UI
- Configuration: AI models (`TEXT_MODEL`, `IMAGE_MODEL`, `AUDIO_MODEL`) and
  notifications (`SLACK_WEBHOOK`, hardcoded username `podcast-generator`) are
  configured via environment variables; removed legacy Ntfy and ComfyUI mounts.
- UI Scope: Frontend Settings page is restricted to user account and security
  management, removing user-level model and notification preferences.
