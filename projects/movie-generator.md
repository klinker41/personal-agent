---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-08-30T11:30:16.339481+00:00
confidence: 0.95
---

# Project: Movie-Generator

- **Models**: Configured with `gemini-3.7-flash` for reasoning and
  orchestration, and `gemini-omni-1.1-flash` for video generation.
- **Rendering & Upscaling**: Hierarchical stitching (chunks -> `scene.mp4` ->
  `movie.mp4`) with resolution-aware generation/upscaling (360p Draft, 720p HD,
  1080p FHD, 4K UHD) that skips already-upscaled chunks.
- **Camera Continuity & Multimodal Prompting**:
  - `backend/src/services/gemini.ts` (`generateSceneChunks`): Evaluates
    `camera_continuity` (`continuous` vs `new_shot`).
  - Continuous takes (split dialogue, tracking shots, sustained two-shots,
    unbroken action) attach prior chunk `video.mp4` as multimodal references
    with temporal directives.
  - Stage 4.8A handles concept plate image prompt generation via
    `generateCameraSetupImagePrompt`.
- **Error Recovery & Prompt Sanitization**:
  - Video generation executes Attempt 1 with raw script prompts and applies
    reactive sanitization (`sanitizeAndFixPrompt` via Gemini Flash) only on
    error or policy rejection.
  - Rewritten prompts are persisted to `prompt.txt` and `chunk_manifest.json`.
- **Response Parsing & Formatting Utilities**:
  - `backend/src/utils/jsonParser.ts` (`extractGeminiResponseText`): Safely
    extracts model text and inspects diagnostic metadata (`finishReason`,
    `safetyRatings`, `blockReason`) when responses are empty.
  - `formatTimelineBeat` and `formatTimelineAndAudio`: Format structured JSON
    timeline beats from LLM outputs to prevent `[object Object]`
    serialization.
