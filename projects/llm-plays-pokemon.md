---
topic: llm-plays-pokemon
category: project
tags: [project, llm-plays-pokemon]
updated_at: 2026-10-16T00:01:13.745140+00:00
---

Notice: The task '5b10366d-be8f-420a-a415-eef99de286d7/task-60' has completed.
Completed At: 2026-09-23T00:35:05Z
The task exited with code 0.
Task output:
Total lines: 111
Lines > 80 chars: 0

- TTS narration architecture introduces a `tts_settings` SQLite table with API
keys encrypted at rest via AES-256-GCM (`enc:v1:` prefix) and a `narration TEXT`
column on the `decisions` table for LLM commentary.
- Broadcast narration audio is handled via `NarrationMixer`, which ducks game
audio to 0.5 gain during playback, resamples audio to output formats using
linear interpolation, and buffers a single non-interrupting pending clip.

- Added optional `narration` (max 200 chars) and `narration_style` (max 40
chars) fields to `PokemonDecisionZodSchema`, utilizing `z.preprocess` for string
truncation to avoid turn validation failures while maintaining compatibility
with `z.toJSONSchema`.
- Integrated narration context (`narrationEnabled`, `previousNarrations`) into
`DecisionContext`, `SystemPromptContext`, and `campaign_memory` (`TurnReceipt`),
ensuring system prompts remain byte-identical when narration is disabled.

- Added admin TTS configuration endpoints (`/tts`, `/tts/voice`, `/tts/test`)
and UI supporting write-only API key validation, voice design caching via source
hashing, upstream 502 error sanitization to prevent credential leaks, and
narrator failure resets.

- TTS settings in AppDatabase conditionally encrypt API keys using
STREAM_ENCRYPTION_KEY when present, falling back to plaintext SQLite storage
when stream encryption is not configured.
- Admin TTS endpoints (/api/admin/tts/voice and /api/admin/tts/test) accept form
input overrides (api_key, endpoint_url, model_name) to allow testing credentials
before saving.
