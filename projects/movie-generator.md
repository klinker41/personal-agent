---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-09-28T00:32:53.366348+00:00
confidence: 0.95
---

# Project: Movie-Generator

## Architecture & Access Control
- **Stack & Assets**: Built on `oven/bun:alpine` with Hono and CLI `ffmpeg`,
  using native `Bun.password`, `bun test`, and `app.request()`. Route
  `/assets/*` prevents path-traversal, serving `generated_assets/` with
  fallback to `frontend/dist/assets/`.
- **RBAC & Ownership**: Enforces movie isolation via `created_by === user.id`.
  Admin rights require both `"role": "admin"` and `"is_admin": true` in
  `data/users.json` (unspecified or legacy accounts default to standard user).

## Generation Pipeline & Continuity
- **Pipeline & Tiers**: Orchestrated by `gemini-3.7-flash` with
  `gemini-omni-1.1-flash` video generation. Stitches chunks into `scene.mp4`
  and `movie.mp4` across four tiers (360p Draft, 720p HD, 1080p FHD, 4K UHD),
  skipping existing or upscaled chunks.
- **Continuity & Plates**: Stage 4.8A creates concept plate prompts via
  `generateCameraSetupImagePrompt`. In `backend/src/services/gemini.ts`,
  `generateSceneChunks` evaluates `camera_continuity` (`continuous` vs
  `new_shot`); continuous takes attach prior chunk `video.mp4` with temporal
  cues as multimodal references.

## Safety, Diagnostics & Serialization
- **Safety & Sanitization**: `DEFAULT_SAFETY_SETTINGS` sets `BLOCK_ONLY_HIGH`
  across all categories, including `HARM_CATEGORY_CIVIC_INTEGRITY`. Raw prompt
  failures or `PROHIBITED_CONTENT` trigger Flash sanitization
  (`sanitizeSceneContent` / `sanitizeAndFixPrompt`), writing revised `setting`
  and `action_summary` to `prompt.txt` and `chunk_manifest.json`.
- **Diagnostics & Formatting**: On empty responses, `extractGeminiResponseText`
  (`backend/src/utils/jsonParser.ts`) inspects `finishReason`, `safetyRatings`,
  and `blockReason`. `formatTimelineBeat` and `formatTimelineAndAudio`
  explicitly format timeline beats to prevent `[object Object]` serialization.
