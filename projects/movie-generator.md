---
topic: movie-generator
category: project
tags: [project, movie-generator]
updated_at: 2026-08-29T11:59:13.144689+00:00
confidence: 0.95
---

# Project: Movie-Generator

- **Models**: Configured with `gemini-3.7-flash` for text reasoning and
  orchestration, and `gemini-omni-1.1-flash` for video generation.
- **Rendering & Upscaling Pipeline**: Uses a hierarchical stitching workflow
  (individual chunks -> `scene.mp4` -> `movie.mp4`) with a resolution-aware
  generation and upscale process (360p Draft, 720p HD, 1080p Full HD,
  4K Ultra HD) that skips already-upscaled chunks.
- **Camera Continuity & Chunking**: `backend/src/services/gemini.ts`
  (`generateSceneChunks`) defines heuristics for `camera_continuity`
  (`continuous` vs `new_shot`). For continuous takes (split dialogue across
  chunks, dynamic tracking shots, sustained two-shots, unbroken action),
  previous chunk `video.mp4` files are attached as multimodal references to
  Gemini Omni prompts with temporal continuity directives.
- **Prompting & Error Recovery**:
  - Stage 4.8A handles camera setup concept plate image prompt generation via
    `generateCameraSetupImagePrompt`.
  - Video generation executes Attempt 1 with raw script prompts and applies
    reactive sanitization (`sanitizeAndFixPrompt` via Gemini Flash) only on
    error or policy rejection, persisting rewritten prompts to `prompt.txt`
    and `chunk_manifest.json`.
- **Response Parsing & Formatting Utilities**:
  - `backend/src/utils/jsonParser.ts` (`extractGeminiResponseText`): Safely
    extracts model text and inspects diagnostic metadata (`finishReason`,
    `safetyRatings`, `blockReason`) when responses are empty.
  - `formatTimelineBeat` and `formatTimelineAndAudio`: Format structured JSON
    timeline beats from LLM outputs to prevent `[object Object]`
    serialization.
