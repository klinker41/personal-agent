---
topic: llm-plays-pokemon
category: project
tags: [project, llm-plays-pokemon]
updated_at: 2026-10-18T06:43:21.878919+00:00
---

# Project: LLM-Plays-Pokemon

## TTS Narration & Audio Architecture
- **Audio Mixing & Storage**: `NarrationMixer` manages broadcast narration,
  ducking game audio to 0.5 gain during playback, resampling via linear
  interpolation, and buffering a single non-interrupting clip. `AppDatabase`
  stores API keys in `tts_settings` encrypted at rest via AES-256-GCM
  (`enc:v1:` prefix) when `STREAM_ENCRYPTION_KEY` is configured (falling back
  to plaintext), and records commentary in the `decisions` table
  `narration TEXT` column.

## Decision Schema & Context Integration
- **Validation & Context Flow**: `PokemonDecisionZodSchema` defines optional
  `narration` (max 200 chars) and `narration_style` (max 40 chars), using
  `z.preprocess` string truncation to prevent validation failures while
  maintaining `z.toJSONSchema` compatibility. Turn context
  (`narrationEnabled`, `previousNarrations`) integrates into
  `DecisionContext`, `SystemPromptContext`, and `campaign_memory`
  (`TurnReceipt`), keeping system prompts byte-identical when narration is
  disabled.

## Admin Configuration & Endpoints
- **Admin API & Security**: Endpoints (`/tts`, `/tts/voice`, `/tts/test`
  under `/api/admin/tts`) and UI support write-only API key validation,
  voice design caching via source hashing, upstream 502 error sanitization
  to prevent credential leaks, narrator failure resets, and form input
  overrides (`api_key`, `endpoint_url`, `model_name`) to test credentials
  prior to saving.
