---
topic: gemini-speech-generation
category: knowledge
tags: [knowledge, gemini-speech-generation]
updated_at: 2026-10-04T00:00:46.110547+00:00
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

- Gemini Interactions API (`/v1beta/interactions`) supports speech generation
via `response_format: { type: 'audio' }` with optional `speech_metadata` style
annotations; audio data is returned base64-encoded in the last model output step
content as either 16-bit 24kHz mono WAV or headerless little-endian L16 PCM
(`audio/l16`).
- Gemini prompted voice design endpoint (`POST /v1beta/voices`) creates custom
synthetic voices from descriptions using `{ store: true, voice: { model, type:
'prompted', prompted: { input: description } } }`, returning a voice ID and
base64 sample audio.
