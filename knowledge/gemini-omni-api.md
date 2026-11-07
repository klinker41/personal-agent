---
topic: gemini-omni-api
category: knowledge
tags: [knowledge, gemini, gemini-omni, video, safety, sanitization]
updated_at: 2026-11-07T07:31:37.958422+00:00
compacted_at: 2026-11-07T07:31:37.958422+00:00
confidence: 0.95
---

# Knowledge: Gemini Omni Video Generation, Safety & Prompt Sanitization

## Models & Capabilities

- **`gemini-omni-1.1-flash`**: The stable video generation model in the
  Generative Language `v1beta` REST API. It supports explicit `response_format`
  resolutions (`360p`, `720p`, `1080p`, `4K`), first/last frame keyframing, and
  up to 10 seconds of multimodal reference video (`video/mp4`) to preserve
  character appearance, velocity, and camera motion continuity across shots.
- **`gemini-omni-1.2-flash`**: A joint audio-visual co-generation model. It
  does not support driving lip-sync animations from pre-recorded dialogue audio
  tracks, and the API currently rejects uploaded audio inputs or voice timbre
  references.

## Safety Filtering & Response Validation

- **Empty Outputs & Safety Diagnostics**: When output is filtered (`SAFETY`,
  `BLOCKLIST`, `PROHIBITED_CONTENT`, `OTHER`), `response.text` may be empty and
  `inlineData` media payloads omitted without throwing errors. Always inspect
  `candidate.finishReason`, `safetyRatings`, and `promptFeedback.blockReason`,
  and handle missing media parts defensively.
- **Gateway-Level Prompt Filtering**: A `promptFeedback.blockReason` of
  `PROHIBITED_CONTENT` with zero candidates indicates pre-generation gateway
  filtering, often triggered by combining trademarked franchise names with
  combat or violence descriptors. Avoid explicit trigger words (e.g., "blood",
  graphic gore) in system instructions to preserve PG-13 action pacing without
  tripping gateway filters.
- **Candidate Parsing**: When thinking is enabled, iterate and concatenate all
  non-thought text parts rather than indexing
  `candidates[0].content.parts[0].text`. Reject responses ending in
  `MAX_TOKENS` (truncation) or `SAFETY`, and enforce
  `responseMimeType: 'application/json'` for structured outputs.
- **Safety Configuration**: Setting `BLOCK_ONLY_HIGH` in `@google/genai`
  requires configuring all five harm categories, including
  `HarmCategory.HARM_CATEGORY_CIVIC_INTEGRITY`.

## Prompt Sanitization & Error Recovery

- **Reactive Sanitization**: Execute raw natural prompts first without
  preprocessing. On safety or `PROHIBITED_CONTENT` rejections, invoke an LLM
  compliance rewriter to replace trademarked franchise lore and de-escalate
  violent action with generic equivalents before retrying.
- **Diagnostic Reprompting & Telemetry**: Recover from validation failures by
  passing structured diagnostics (syntax errors, schema boundary failures) and
  sanitized response snippets rather than raw unparsed payloads. Preserve
  per-attempt history, raw responses, and diagnostic reprompts across all retry
  cycles (on both success and exhaustion) to support post-mortem analysis.

## Scene Chunking & Post-Processing

- **Camera Continuity vs. Cuts**: Under shot/reverse-shot coverage rules, scene
  planners default to camera cuts (`new_shot`) when dialogue alternates between
  characters. Require explicit heuristics for continuous takes
  (`camera_continuity: 'continuous'`), reserving them for tracking movement,
  sustained shared staging, or single-character dialogue exceeding per-chunk
  limits (e.g., 10 seconds or 18–20 words).
- **FFmpeg Concatenation**: Use quote escaping and unique temporary file paths
  for FFmpeg concat demuxer lists, paired with a re-encode fallback to handle
  mismatched stream parameters across generated chunks.
