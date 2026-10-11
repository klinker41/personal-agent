---
topic: media-generator
category: project
tags: [project, media-generator]
updated_at: 2026-10-11T00:41:44.984960+00:00
---

# Project: Media-Generator

## Model Configuration & Selection
- **Centralized Model IDs:** Default Gemini and Lyria model constants are
  centralized in `shared/models.ts` and mirrored in `.env.example`, stripping
  the `models/` prefix:
  - `DEFAULT_TEXT_MODEL`: `gemini-3.8-flash` (text)
  - `DEFAULT_IMAGE_MODEL`: `gemini-3-pro-image` (image)
  - `DEFAULT_AUDIO_MODEL`: `gemini-3.8-flash-tts` (audio)
  - `DEFAULT_VIDEO_MODEL`: `gemini-omni-1.1-flash` (video)
  - `DEFAULT_MUSIC_MODEL`: `lyria-3-clip-preview` (music)
- **Selection Criteria:** Project defaults restrict models to Gemini variants
  (or Lyria for music) and permit preview models, while explicitly excluding
  experimental (`exp`, `experimental`) and `latest` aliases.
