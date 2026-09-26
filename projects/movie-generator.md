---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-09-26T00:34:02.825981+00:00
confidence: 0.95
---

# Project: Movie-Generator

## Architecture & Access Control
- **Stack & Assets**: Built on `oven/bun:alpine` with Hono and CLI `ffmpeg`,
  using native `Bun.password`, `bun test`, and `app.request()`. Route
  `/assets/*` enforces path-traversal protection, serving
  `generated_assets/` with fallback to `frontend/dist/assets/`.
- **RBAC & Ownership**: Restricts movie access via `created_by === user.id`.
  Admin access strictly requires both `"role": "admin"` and
  `"is_admin": true` in `data/users.json` (legacy or unspecified accounts
  default to standard).

## Generation Pipeline & Continuity
- **Pipeline & Tiers**: Orchestrated by `gemini-3.7-flash` with
  `gemini-omni-1.1-flash` video generation. Stitches `chunks` ->
  `scene.mp4` -> `movie.mp4` across tiers (360p Draft, 720p HD, 1080p FHD,
  4K UHD), skipping existing or upscaled chunks.
- **Continuity & Plates**: Stage 4.8A generates concept plate prompts via
  `generateCameraSetupImagePrompt`. In `backend/src/services/gemini.ts`,
  `generateSceneChunks` evaluates `camera_continuity` (`continuous` vs
  `new_shot`); continuous takes attach the prior chunk's `video.mp4` with
  temporal cues as multimodal references.

## Safety, Diagnostics & Serialization
- **Safety & Sanitization**: `DEFAULT_SAFETY_SETTINGS` sets
  `BLOCK_ONLY_HIGH` across all categories, including
  `HARM_CATEGORY_CIVIC_INTEGRITY`. Prompts run raw first; failures or
  `PROHIBITED_CONTENT` trigger Flash sanitization (`sanitizeSceneContent` /
  `sanitizeAndFixPrompt`), saving revised `setting` and `action_summary` to
  `prompt.txt` and `chunk_manifest.json`.
- **Diagnostics & Formatting**: On empty responses,
  `extractGeminiResponseText` (`backend/src/utils/jsonParser.ts`) inspects
  `finishReason`, `safetyRatings`, and `blockReason`. `formatTimelineBeat`
  and `formatTimelineAndAudio` explicitly format timeline beats to prevent
  `[object Object]` serialization.
