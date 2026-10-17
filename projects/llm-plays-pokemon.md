---
topic: llm-plays-pokemon
category: project
tags: [project, llm-plays-pokemon]
updated_at: 2026-10-17T00:40:51.403022+00:00
---

# Project: LLM-Plays-Pokemon

## TTS Narration & Audio Architecture
- Broadcast narration audio is managed by `NarrationMixer`, which ducks game
  audio to 0.5 gain during playback, resamples audio to output formats using
  linear interpolation, and buffers a single non-interrupting pending clip.
- SQLite table `tts_settings` in `AppDatabase` stores API keys encrypted at
  rest via AES-256-GCM (`enc:v1:` prefix) when `STREAM_ENCRYPTION_KEY` is
  configured, falling back to plaintext storage if unset. The `decisions` table
  includes a `narration TEXT` column for LLM commentary.

## Decision Schema & Context Integration
- `PokemonDecisionZodSchema` defines optional `narration` (max 200 chars) and
  `narration_style` (max 40 chars) fields, using `z.preprocess` for string
  truncation to prevent turn validation failures while maintaining
  compatibility with `z.toJSONSchema`.
- Narration context (`narrationEnabled`, `previousNarrations`) is integrated
  into `DecisionContext`, `SystemPromptContext`, and `campaign_memory`
  (`TurnReceipt`), ensuring system prompts remain byte-identical when narration
  is disabled.

## Admin Configuration & Endpoints
- Admin TTS configuration endpoints (`/tts`, `/tts/voice`, `/tts/test` under
  `/api/admin/tts`) and UI support write-only API key validation, voice design
  caching via source hashing, upstream 502 error sanitization to prevent
  credential leaks, narrator failure resets, and form input overrides
  (`api_key`, `endpoint_url`, `model_name`) to test credentials prior to saving.
