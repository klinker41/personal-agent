---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-09-30T00:33:37.505338+00:00
confidence: 0.95
---

# Project: Movie-Generator

## Architecture & Access Control
- **Stack & Routing**: Built on `oven/bun:alpine` with Hono, CLI `ffmpeg`,
  native `Bun.password`, `bun test`, and `app.request()`. Route `/assets/*`
  prevents path traversal, serving `generated_assets/` with fallback to
  `frontend/dist/assets/`.
- **RBAC & Isolation**: Enforces movie isolation via `created_by === user.id`.
  Admin access requires both `"role": "admin"` and `"is_admin": true` in
  `data/users.json` (defaults to standard user).

## Generation Pipeline & Continuity
- **Pipeline & Tiers**: Orchestrated by `gemini-3.7-flash` with video generation
  via `gemini-omni-1.1-flash`. Stitches chunks into `scene.mp4` and `movie.mp4`
  across 4 tiers (360p Draft, 720p HD, 1080p FHD, 4K UHD), skipping existing
  or upscaled chunks.
- **Continuity & Plates**: Stage 4.8A generates concept plate prompts using
  `generateCameraSetupImagePrompt`. In `backend/src/services/gemini.ts`,
  `generateSceneChunks` handles `camera_continuity` (`continuous` vs
  `new_shot`); continuous shots attach the prior `video.mp4` with temporal
  cues as multimodal references.

## Safety, Diagnostics & Serialization
- **Safety & Sanitization**: Sets `BLOCK_ONLY_HIGH` across all categories in
  `DEFAULT_SAFETY_SETTINGS` (including `HARM_CATEGORY_CIVIC_INTEGRITY`). Prompt
  failures or `PROHIBITED_CONTENT` trigger Flash sanitization
  (`sanitizeSceneContent` / `sanitizeAndFixPrompt`), writing revised `setting`
  and `action_summary` to `prompt.txt` and `chunk_manifest.json`.
- **Diagnostics & Formatting**: `extractGeminiResponseText` in
  `backend/src/utils/jsonParser.ts` inspects `finishReason`, `safetyRatings`,
  and `blockReason` on empty outputs. `formatTimelineBeat` and
  `formatTimelineAndAudio` explicitly format timeline beats to prevent
  `[object Object]` serialization.
