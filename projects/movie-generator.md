---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-10-07T00:35:13.098516+00:00
confidence: 0.95
---

# Project: Movie-Generator

## Architecture & Access Control
- **Stack & Routing**: `oven/bun:alpine` with Hono, CLI `ffmpeg`, native
  `Bun.password`, and `bun test` via `app.request()`. Route `/assets/*`
  guards against path traversal, serving `generated_assets/` with fallback to
  `frontend/dist/assets/`.
- **Tenant Isolation & RBAC**: Enforces tenant isolation via
  `created_by === user.id`. Admin access strictly requires both
  `"role": "admin"` and `"is_admin": true` in `data/users.json` (defaults to
  standard user).

## Generation Pipeline & Continuity
- **Pipeline & Tiers**: Orchestrated by `gemini-3.7-flash` with video
  generation via `gemini-omni-1.1-flash`. Stitches chunks into `scene.mp4` and
  `movie.mp4` across 4 resolution tiers (360p Draft, 720p HD, 1080p FHD,
  4K UHD), skipping existing or upscaled chunks.
- **Continuity & Plates**: Stage 4.8A generates concept plate prompts via
  `generateCameraSetupImagePrompt`. In `backend/src/services/gemini.ts`,
  `generateSceneChunks` evaluates `camera_continuity` (`continuous` vs
  `new_shot`); continuous shots attach prior `video.mp4` with temporal cues as
  multimodal references.

## Safety, Diagnostics & Serialization
- **Safety & Sanitization**: Sets `BLOCK_ONLY_HIGH` across all categories in
  `DEFAULT_SAFETY_SETTINGS` (including `HARM_CATEGORY_CIVIC_INTEGRITY`). Prompt
  failures or `PROHIBITED_CONTENT` trigger Flash sanitization
  (`sanitizeSceneContent` / `sanitizeAndFixPrompt`), writing revised `setting`
  and `action_summary` to `prompt.txt` and `chunk_manifest.json`.
- **Diagnostics & Formatting**: In `backend/src/utils/jsonParser.ts`,
  `extractGeminiResponseText` inspects `finishReason`, `safetyRatings`, and
  `blockReason` on empty outputs. `formatTimelineBeat` and
  `formatTimelineAndAudio` explicitly format timeline beats to prevent
  `[object Object]` serialization.
