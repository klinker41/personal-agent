---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-10-17T00:36:43.232482+00:00
confidence: 0.95
---

# Project: Movie-Generator

## Architecture & Security
- **Runtime Stack & Routing**: Containerized on `oven/bun:alpine` with Hono,
  CLI `ffmpeg`, `Bun.password`, and `bun test` (`app.request()`). Route
  `/assets/*` guards against directory traversal, serving `generated_assets/`
  with fallback to `frontend/dist/assets/`.
- **Tenant Isolation & RBAC**: Boundary checks enforce
  `created_by === user.id`. Admin rights strictly require both
  `"role": "admin"` and `"is_admin": true` in `data/users.json` (defaults to
  standard user).

## Video Generation & Continuity
- **Pipeline & Tiers**: Orchestrated by `gemini-3.7-flash` with video
  generation via `gemini-omni-1.1-flash`. Stitches chunks into `scene.mp4` and
  `movie.mp4` across 4 tiers (360p Draft, 720p HD, 1080p FHD, 4K UHD),
  skipping existing or upscaled chunks.
- **Shot Continuity & Plates**: Stage 4.8A creates concept plates with
  `generateCameraSetupImagePrompt`. In `backend/src/services/gemini.ts`,
  `generateSceneChunks` branches on `camera_continuity` (`continuous` vs
  `new_shot`), attaching prior `video.mp4` with temporal cues as multimodal
  references for continuous shots.

## Safety, Diagnostics & Serialization
- **Safety & Sanitization**: Sets `BLOCK_ONLY_HIGH` across all categories in
  `DEFAULT_SAFETY_SETTINGS` (including `HARM_CATEGORY_CIVIC_INTEGRITY`).
  Blocked prompts or `PROHIBITED_CONTENT` trigger Flash sanitization
  (`sanitizeSceneContent` / `sanitizeAndFixPrompt`), saving updated `setting`
  and `action_summary` to `prompt.txt` and `chunk_manifest.json`.
- **Diagnostics & Formatting**: In `backend/src/utils/jsonParser.ts`,
  `extractGeminiResponseText` checks `finishReason`, `safetyRatings`, and
  `blockReason` on empty outputs. `formatTimelineBeat` and
  `formatTimelineAndAudio` explicitly format timeline beats to prevent
  `[object Object]` serialization.
