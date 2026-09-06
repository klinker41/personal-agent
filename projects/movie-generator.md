---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-09-06T00:31:33.745512+00:00
confidence: 0.95
---

# Project: Movie-Generator

- **Stack & Architecture**:
  - Backend runs Bun and Hono (`oven/bun:alpine`), replacing Express, dotenv,
    bcrypt (via `Bun.password`), and jest/supertest (via `bun test` and Hono
    `app.request()`).
  - Direct CLI execution of `ffmpeg` replaces `fluent-ffmpeg`.
  - Static route `/assets/*` serves media from `generated_assets/` first with
    fallback to `frontend/dist/assets/`, secured by path traversal checks.
- **Models**: `gemini-3.7-flash` for reasoning and orchestration;
  `gemini-omni-1.1-flash` for video generation.
- **Rendering & Pipeline**:
  - Hierarchical stitching (`chunks` -> `scene.mp4` -> `movie.mp4`) across
    tiers (360p Draft, 720p HD, 1080p FHD, 4K UHD), skipping already-upscaled
    chunks.
  - Stage 4.8A generates concept plate image prompts via
    `generateCameraSetupImagePrompt`.
- **Camera Continuity**: `backend/src/services/gemini.ts`
  (`generateSceneChunks`) evaluates `camera_continuity` (`continuous` vs
  `new_shot`). Continuous takes (split dialogue, tracking shots, sustained
  two-shots, unbroken action) attach the prior chunk's `video.mp4` as a
  multimodal reference with temporal cues.
- **Safety & Error Recovery**:
  - Uses `DEFAULT_SAFETY_SETTINGS` set to `BLOCK_ONLY_HIGH` across all harm
    categories (including `HARM_CATEGORY_CIVIC_INTEGRITY`) on all Gemini
    `generateContent` calls.
  - Attempts raw script prompts first; applies reactive sanitization
    (`sanitizeSceneContent` / `sanitizeAndFixPrompt` via Gemini Flash) only
    on error or `PROHIBITED_CONTENT` blocks.
  - Returns distinct `setting` and `action_summary` fields and persists
    rewritten prompts to `prompt.txt` and `chunk_manifest.json`.
- **Response Parsing & Formatting**:
  - `backend/src/utils/jsonParser.ts` (`extractGeminiResponseText`): Safely
    extracts text and inspects diagnostics (`finishReason`, `safetyRatings`,
    `blockReason`) when responses are empty.
  - `formatTimelineBeat` & `formatTimelineAndAudio`: Formats structured JSON
    timeline beats from LLM outputs to prevent `[object Object]`
    serialization.
