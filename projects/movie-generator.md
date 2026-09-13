---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-09-13T00:31:45.611892+00:00
confidence: 0.95
---

# Project: Movie-Generator

## Architecture & Media Serving
- **Stack & Asset Serving**: Built on `oven/bun:alpine` with Hono and CLI
  `ffmpeg`, using native `Bun.password`, `bun test`, and `app.request()`
  (replacing Express, dotenv, bcrypt, Jest/Supertest, and `fluent-ffmpeg`).
  Traversal-secured `/assets/*` serves `generated_assets/` with fallback to
  `frontend/dist/assets/`.

## Generation Pipeline & Continuity
- **Models & Hierarchical Rendering**: Uses `gemini-3.7-flash` for
  orchestration and `gemini-omni-1.1-flash` for video generation.
  Hierarchically stitches `chunks` -> `scene.mp4` -> `movie.mp4` across
  quality tiers (360p Draft, 720p HD, 1080p FHD, 4K UHD), skipping
  already-upscaled chunks.
- **Plates & Camera Continuity**: Stage 4.8A generates concept plate prompts
  via `generateCameraSetupImagePrompt`. In `backend/src/services/gemini.ts`,
  `generateSceneChunks` evaluates `camera_continuity` (`continuous` vs
  `new_shot`); continuous takes (tracking, two-shots, split dialogue,
  unbroken action) attach the prior chunk's `video.mp4` with temporal cues as
  a multimodal reference.

## Safety, Diagnostics & Serialization
- **Safety & Reactive Sanitization**: Sets `DEFAULT_SAFETY_SETTINGS` to
  `BLOCK_ONLY_HIGH` across all categories (including
  `HARM_CATEGORY_CIVIC_INTEGRITY`) on Gemini calls. Raw prompts run first;
  errors or `PROHIBITED_CONTENT` trigger reactive Flash sanitization
  (`sanitizeSceneContent` / `sanitizeAndFixPrompt`), writing rewritten
  `setting` and `action_summary` to `prompt.txt` and `chunk_manifest.json`.
- **Diagnostics & Formatting**: In `backend/src/utils/jsonParser.ts`,
  `extractGeminiResponseText` inspects diagnostics (`finishReason`,
  `safetyRatings`, `blockReason`) on empty returns. `formatTimelineBeat`
  and `formatTimelineAndAudio` format JSON timeline beats to prevent
  `[object Object]` serialization.

## Access Control & Permissions
- **Role-Based Access Control (RBAC)**: Movies record creator IDs in
  `created_by`. Standard users can only view their own movies
  (`created_by === user.id`), whereas admins (`role: 'admin'` or
  `is_admin: true` in `data/users.json`) have full access. Legacy or
  unspecified accounts default to standard access; admin rights require
  explicitly setting `"role": "admin"` and `"is_admin": true`.
