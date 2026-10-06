---
topic: lyria-music-generation
category: knowledge
tags: [knowledge, lyria-music-generation]
updated_at: 2026-10-06T00:00:48.330915+00:00
confidence: 0.95
---

# Knowledge: Lyria-Music-Generation

- Google Lyria 3 Clip (`lyria-3-clip-preview`) produces 30-second 44.1 kHz
  stereo MP3 audio clips via the Google GenAI Interactions API
  (`https://generativelanguage.googleapis.com/v1beta/interactions` or
  `@google/genai`) using the standard `generateContent` method.
- Lyria models are cataloged alongside Gemini models under the
  `models/lyria-*` namespace (e.g., `models/lyria-3-clip-preview`) via the
  Generative Language REST API (`GET /v1beta/models?pageSize=200`) with
  specific `supportedGenerationMethods`.
- Directional prompt tailoring allows generating cohesive paired tracks
  (e.g., an intro motif with energetic lead-in vs. an outro with reflective
  resolution fading into silence) from a single thematic description.
- When assembling Lyria-generated music with FFmpeg, trimming outro clips
  prematurely cuts off natural resolving chords and musical codas; preserving
  the full 30s duration with a tail safety fade
  (`atrim=0:30,afade=t=in:ss=0:d=1,afade=t=out:st=27:d=3`) maintains
  musicality while preventing waveform pops or clicks.

- Lyria models (such as `lyria-3-clip-preview`) can be listed and discovered via the Gemini REST API (`https://generativelanguage.googleapis.com/v1beta/models`) alongside Gemini models.

- Lyria music generation models (e.g. lyria-3-clip-preview) are discoverable
directly via the Gemini REST API v1beta/models endpoint under models/lyria-*.
