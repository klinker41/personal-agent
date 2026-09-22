---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-09-22T00:33:34.487188+00:00
confidence: 0.95
---

# Project: Movie-Generator

## Architecture & Media Delivery
- **Stack & Assets**: Built on `oven/bun:alpine` with Hono and CLI `ffmpeg`,
  leveraging native `Bun.password`, `bun test`, and `app.request()`. Route
  `/assets/*` enforces path-traversal protection, serving `generated_assets/`
  with fallback to `frontend/dist/assets/`.
- **RBAC & Ownership**: Movies record creator IDs in `created_by`; standard
  users access only their own movies (`created_by === user.id`). Admin access
  requires both `"role": "admin"` and `"is_admin": true` in `data/users.json`,
  defaulting legacy or unspecified accounts to standard access.

## Generation Pipeline & Continuity
- **Pipeline & Quality Tiers**: Orchestrated by `gemini-3.7-flash` with
  `gemini-omni-1.1-flash` video generation. Stitches `chunks` -> `scene.mp4` ->
  `movie.mp4` across quality tiers (360p Draft, 720p HD, 1080p FHD, 4K UHD),
  skipping existing or upscaled chunks.
- **Continuity & Concept Plates**: Stage 4.8A creates concept plate prompts via
  `generateCameraSetupImagePrompt`. `generateSceneChunks`
  (`backend/src/services/gemini.ts`) evaluates `camera_continuity`
  (`continuous` vs `new_shot`); continuous takes (tracking, two-shots, split
  dialogue, unbroken action) attach the prior chunk's `video.mp4` with temporal
  cues as a multimodal reference.

## Safety, Diagnostics & Serialization
- **Safety & Reactive Sanitization**: `DEFAULT_SAFETY_SETTINGS` sets
  `BLOCK_ONLY_HIGH` across all categories, including
  `HARM_CATEGORY_CIVIC_INTEGRITY`. Prompts execute raw first; failures or
  `PROHIBITED_CONTENT` trigger Flash sanitization (`sanitizeSceneContent` /
  `sanitizeAndFixPrompt`), persisting revised `setting` and `action_summary` to
  `prompt.txt` and `chunk_manifest.json`.
- **Diagnostics & Serialization**: On empty responses,
  `extractGeminiResponseText` (`backend/src/utils/jsonParser.ts`) inspects
  `finishReason`, `safetyRatings`, and `blockReason`. `formatTimelineBeat` and
  `formatTimelineAndAudio` explicitly format timeline beats to prevent
  `[object Object]` serialization.
