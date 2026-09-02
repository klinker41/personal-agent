---
topic: podcast-generator
category: project
tags: [project, podcast-generator]
updated_at: 2026-09-02T00:00:11.177070+00:00
confidence: 0.95
---

# Project: Podcast-Generator

- Consolidated from a 6-container setup (PostgreSQL, Redis, Celery worker/beat,
FastAPI, Nginx) into a single Node.js/TypeScript container serving the React SPA
and REST API.
- Replaced PostgreSQL with filesystem JSON persistence in data/users.json,
data/podcasts/, and data/outputs/.
- Replaced Redis and Celery with an in-process concurrency-limited JobQueue and
PodcastScheduler.
- Pipeline generates content via Gemini Text, Gemini Image (1:1 cover art),
Gemini TTS multi-speaker audio synthesis, and FFmpeg audio assembly with ID3v2
tagging and Jellyfin metadata sync.
- Notifications standardized on Slack webhooks (configured via SLACK_WEBHOOK
environment variable or user settings); removed legacy Ntfy support and ComfyUI
input folder mounts.
