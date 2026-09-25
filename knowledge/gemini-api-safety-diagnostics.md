---
topic: gemini-api-safety-diagnostics
category: knowledge
tags: [knowledge, gemini-api-safety-diagnostics]
updated_at: 2026-09-25T00:37:03.043267+00:00
confidence: 0.95
---

# Knowledge: Gemini-Api-Safety-Diagnostics

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
