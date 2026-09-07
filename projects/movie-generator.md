---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-09-07T00:32:53.810122+00:00
confidence: 0.95
---

# Project: Movie-Generator

## Architecture & Media Serving
- **Bun & Hono Backend**: Runs on `oven/bun:alpine`, replacing Express,
  dotenv, bcrypt (via `Bun.password`), and Jest/Supertest (via `bun test`
  and Hono `app.request()`).
- **Media Processing & Asset Delivery**: Direct CLI execution of `ffmpeg`
  replaces `fluent-ffmpeg`. Route `/assets/*` serves `generated_assets/`
  first with fallback to `frontend/dist/assets/`, secured by path traversal
  checks.

## Generation Pipeline & Continuity
- **Models**: Uses `gemini-3.7-flash` for reasoning and orchestration;
  `gemini-omni-1.1-flash` for video generation.
- **Hierarchical Rendering**: Stitches `chunks` -> `scene.mp4` -> `movie.mp4`
  across tiers (360p Draft, 720p HD, 1080p FHD, 4K UHD), skipping
  already-upscaled chunks.
- **Concept Plates & Continuity**: Stage 4.8A generates concept plate image
  prompts via `generateCameraSetupImagePrompt`.
  `backend/src/services/gemini.ts` (`generateSceneChunks`) evaluates
  `camera_continuity` (`continuous` vs `new_shot`); continuous takes (split
  dialogue, tracking shots, sustained two-shots, unbroken action) attach the
  prior chunk's `video.mp4` as a multimodal reference with temporal cues.

## Safety, Diagnostics & Error Recovery
- **Safety Policy**: Sets `DEFAULT_SAFETY_SETTINGS` to `BLOCK_ONLY_HIGH`
  across all harm categories (including `HARM_CATEGORY_CIVIC_INTEGRITY`) on
  all Gemini `generateContent` calls.
- **Reactive Prompt Sanitization**: Attempts raw script prompts first,
  applying reactive sanitization (`sanitizeSceneContent` /
  `sanitizeAndFixPrompt` via Gemini Flash) only on error or
  `PROHIBITED_CONTENT` blocks. Returns distinct `setting` and
  `action_summary` fields, persisting rewrites to `prompt.txt` and
  `chunk_manifest.json`.
- **Response Diagnostics & Formatting**: `backend/src/utils/jsonParser.ts`
  (`extractGeminiResponseText`) safely extracts text and inspects diagnostics
  (`finishReason`, `safetyRatings`, `blockReason`) when responses are empty.
  `formatTimelineBeat` and `formatTimelineAndAudio` format structured JSON
  timeline beats from LLM outputs to prevent `[object Object]` serialization.
