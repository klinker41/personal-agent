---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-09-17T00:32:02.827911+00:00
confidence: 0.95
---

# Project: Movie-Generator

## Architecture & Media Serving
- **Stack & Assets**: Built on `oven/bun:alpine` with Hono and CLI `ffmpeg`,
  using native `Bun.password`, `bun test`, and `app.request()`.
  Traversal-secured `/assets/*` serves `generated_assets/` with fallback to
  `frontend/dist/assets/`.

## Generation Pipeline & Continuity
- **Orchestration & Rendering**: Orchestrated by `gemini-3.7-flash` with
  `gemini-omni-1.1-flash` video generation. Stitches `chunks` -> `scene.mp4` ->
  `movie.mp4` across quality tiers (360p Draft, 720p HD, 1080p FHD, 4K UHD),
  skipping existing or upscaled chunks.
- **Plates & Camera Continuity**: Stage 4.8A creates concept plate prompts via
  `generateCameraSetupImagePrompt`. In `backend/src/services/gemini.ts`,
  `generateSceneChunks` checks `camera_continuity` (`continuous` vs `new_shot`);
  continuous takes (tracking, two-shots, split dialogue, unbroken action)
  attach the previous chunk's `video.mp4` with temporal cues as a
  multimodal reference.

## Safety, Diagnostics & Serialization
- **Safety & Reactive Sanitization**: `DEFAULT_SAFETY_SETTINGS` is set to
  `BLOCK_ONLY_HIGH` across all categories (including
  `HARM_CATEGORY_CIVIC_INTEGRITY`). Prompts run raw first; failures or
  `PROHIBITED_CONTENT` trigger reactive Flash sanitization
  (`sanitizeSceneContent` / `sanitizeAndFixPrompt`), writing revised
  `setting` and `action_summary` to `prompt.txt` and `chunk_manifest.json`.
- **Diagnostics & Formatting**: In `backend/src/utils/jsonParser.ts`,
  `extractGeminiResponseText` inspects `finishReason`, `safetyRatings`, and
  `blockReason` on empty responses. `formatTimelineBeat` and
  `formatTimelineAndAudio` explicitly format timeline beats to prevent
  `[object Object]` serialization.

## Access Control & Permissions
- **Role-Based Access Control (RBAC)**: Movies record creator IDs in
  `created_by`; standard users can only view their own movies
  (`created_by === user.id`). Admin access requires both `"role": "admin"`
  and `"is_admin": true` in `data/users.json`; legacy or unspecified accounts
  default to standard access.
