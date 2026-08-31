---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-08-31T00:31:40.025159+00:00
confidence: 0.95
---

# Project: Movie-Generator

- **Models**: Uses `gemini-3.7-flash` for reasoning/orchestration and
  `gemini-omni-1.1-flash` for video generation.
- **Rendering & Upscaling**: Hierarchical stitching (`chunks` -> `scene.mp4` ->
  `movie.mp4`) across tiers (360p Draft, 720p HD, 1080p FHD, 4K UHD), skipping
  already-upscaled chunks.
- **Camera Continuity & Prompting**:
  - `backend/src/services/gemini.ts` (`generateSceneChunks`): Evaluates
    `camera_continuity` (`continuous` vs `new_shot`). Continuous takes (split
    dialogue, tracking shots, sustained two-shots, unbroken action) attach the
    previous chunk's `video.mp4` as a multimodal reference with temporal cues.
  - Stage 4.8A generates concept plate image prompts using
    `generateCameraSetupImagePrompt`.
- **Error Recovery & Prompt Sanitization**: Attempts generation first with raw
  script prompts; applies reactive sanitization (`sanitizeAndFixPrompt` via
  Gemini Flash) only on error or policy rejection. Persists rewritten prompts
  to `prompt.txt` and `chunk_manifest.json`.
- **Response Parsing & Formatting**:
  - `backend/src/utils/jsonParser.ts` (`extractGeminiResponseText`): Safely
    extracts model text and checks diagnostic metadata (`finishReason`,
    `safetyRatings`, `blockReason`) when responses are empty.
  - `formatTimelineBeat` & `formatTimelineAndAudio`: Formats structured JSON
    timeline beats from LLM outputs to prevent `[object Object]` serialization.
