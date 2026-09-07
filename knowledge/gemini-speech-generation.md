---
topic: gemini-speech-generation
category: knowledge
tags: [knowledge, gemini-speech-generation]
updated_at: 2026-09-07T00:01:31.432443+00:00
confidence: 0.95
---

# Knowledge: Gemini-Speech-Generation

- The Gemini API speech generation service provides 30 prebuilt voice models
(e.g., Aoede, Umbriel, Puck, Charon, Kore, Fenrir, Zephyr) referenced directly
by voice name.

- Gemini TTS outputs raw 24kHz 16-bit mono PCM audio; packaging it in-memory
with standard RIFF/WAVE headers allows direct HTML5 `<audio>` playback in
browsers without spawning external FFmpeg processes.

- Gemini TTS voice assignment across multi-speaker casts can be automated by
tracking existing speaker profiles and filtering/prioritizing unused voices from
the 30 official Gemini TTS voices to ensure vocal diversity.
