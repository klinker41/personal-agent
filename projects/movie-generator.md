---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-09-09T00:32:17.171050+00:00
confidence: 0.95
---

# Project: Movie-Generator

## Architecture & Media Serving
- **Runtime & Stack**: Built on `oven/bun:alpine` with Hono. Native Bun APIs
  (`Bun.password`, `bun test`, `app.request()`) replace Express, dotenv,
  bcrypt, and Jest/Supertest.
- **Media & Asset Delivery**: CLI `ffmpeg` replaces `fluent-ffmpeg`. Route
  `/assets/*` serves `generated_assets/` with fallback to
  `frontend/dist/assets/`, secured against path traversal.

## Generation Pipeline & Continuity
- **Models & Hierarchical Rendering**: Uses `gemini-3.7-flash` for
  orchestration/reasoning and `gemini-omni-1.1-flash` for video generation.
  Stitches `chunks` -> `scene.mp4` -> `movie.mp4` across quality tiers (360p
  Draft, 720p HD, 1080p FHD, 4K UHD), skipping already-upscaled chunks.
- **Concept Plates & Continuity**: Stage 4.8A generates concept plate image
  prompts via `generateCameraSetupImagePrompt`. In
  `backend/src/services/gemini.ts`, `generateSceneChunks` evaluates
  `camera_continuity` (`continuous` vs `new_shot`); continuous takes
  (tracking shots, sustained two-shots, split dialogue, unbroken action)
  attach prior chunk `video.mp4` with temporal cues as multimodal reference.

## Safety, Diagnostics & Error Recovery
- **Safety & Prompt Sanitization**: `DEFAULT_SAFETY_SETTINGS` is
  `BLOCK_ONLY_HIGH` across all categories (including
  `HARM_CATEGORY_CIVIC_INTEGRITY`) on Gemini calls. Prompts run raw first;
  errors or `PROHIBITED_CONTENT` trigger reactive sanitization
  (`sanitizeSceneContent` / `sanitizeAndFixPrompt` via Gemini Flash), saving
  rewritten `setting` and `action_summary` to `prompt.txt` and
  `chunk_manifest.json`.
- **Response Diagnostics & Formatting**: In `backend/src/utils/jsonParser.ts`,
  `extractGeminiResponseText` inspects diagnostics (`finishReason`,
  `safetyRatings`, `blockReason`) on empty returns. `formatTimelineBeat` and
  `formatTimelineAndAudio` format JSON timeline beats to prevent
  `[object Object]` serialization.
