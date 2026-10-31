---
topic: gemini-omni-api
category: knowledge
tags: [knowledge, gemini, gemini-omni, video, safety, sanitization]
updated_at: 2026-10-30T14:36:10.703208+00:00
compacted_at: 2026-10-30T14:36:10.703208+00:00
confidence: 0.95
---

# Knowledge: Gemini Omni Video Generation, Safety & Prompt Sanitization

- Gemini Omni 1.1 Flash supports explicit resolution configuration in
response_format (360p, 720p, 1080p, 4K) and first/last frame keyframing.
- Gemini Omni 1.1 Flash analyzes up to 10 seconds of attached multimodal
reference video (video/mp4) for maintaining character appearance, velocity, and
camera motion continuity across shots.

- gemini-omni-1.1-flash serves as the stable release model for video generation
workloads queried via the Gemini REST API.

- `gemini-omni-1.1-flash` is available in the Generative Language v1beta API for
video generation tasks.

- **Empty Outputs and Safety Diagnostics**: When Gemini filters output due to
  safety, copyright, or policy constraints (`SAFETY`, `BLOCKLIST`,
  `PROHIBITED_CONTENT`, `OTHER`), `response.text` can return an empty string,
  and media payloads (`inlineData`) may be omitted without throwing errors.
  Client applications should inspect `candidate.finishReason`,
  `safetyRatings`, and `promptFeedback.blockReason` rather than assuming
  successful generation, and defensively handle missing media parts.
- **Gateway-Level Prompt Filtering**: A `promptFeedback.blockReason` of
  `PROHIBITED_CONTENT` with zero candidates indicates pre-generation
  gateway-level input filtering, commonly triggered by concatenated prompts
  combining trademarked franchise names with combat/violence descriptors.
- **Candidate Parsing and Validation**: When parsing responses (especially
  with thinking enabled), iterate and concatenate all non-thought text parts
  rather than indexing `candidates[0].content.parts[0].text`. Reject
  responses terminated by `MAX_TOKENS` (truncation) or `SAFETY`, and enforce
  `responseMimeType: 'application/json'` on structured configurations.
- **Safety Setting Configuration**: In `@google/genai`, configuring
  permissive safety settings via `BLOCK_ONLY_HIGH` requires setting all five
  harm categories, including `HarmCategory.HARM_CATEGORY_CIVIC_INTEGRITY`.

- A reactive prompt sanitization strategy allows raw natural prompts to execute
first without preprocessing, invoking an LLM compliance rewriter only upon
safety or trademark rejection before retrying generation.

- When pipeline steps fail due to non-retryable `PROHIBITED_CONTENT` blocks,
implement automatic recovery using LLM-guided rewriting to replace trademarked
franchise lore and de-escalate violent action with generic equivalents before
retrying.

- Diagnostic reprompting loops achieve reliable error recovery by returning
structured validation diagnostics (syntax errors, schema boundary failures)
alongside sanitized response snippets rather than raw unparsed payloads.

- Preserving per-attempt history, raw response text, and intermediate diagnostic
reprompts across all retry cycles—on both success and exhaustion errors—enables
downstream LLMs to analyze and diagnose validation failure patterns during
post-mortems.

- LLMs planning video scene chunks tend to default entirely to camera cuts when
shot/reverse-shot coverage rules are enforced; they require explicit heuristics
for continuous camera takes (e.g., dialogue continuation, tracking movement,
sustained shared staging) to maintain continuity across sequential chunks.

- Avoid explicit trigger words (e.g., 'blood' and graphic gore references) in
scene chunking system instructions to prevent tripping gateway safety filters
while retaining dramatic PG-13 action pacing.

- Dialogue chunking engines favor camera cuts (`new_shot`) over continuous shots
because shot/reverse-shot rules trigger a new camera setup on character
alternations.
- Continuous shots (`camera_continuity: 'continuous'`) are primarily reserved
for dialogue continuation when a single character's speech exceeds per-chunk
length limits (e.g., 10 seconds or 18–20 words).

- FFmpeg concat demuxer lists require quote escaping and unique temp file paths,
paired with a re-encode fallback to handle mismatched stream parameters across
generated chunks.
