---
topic: media-generator
category: project
tags: [project, media-generator]
updated_at: 2026-10-10T00:00:54.971879+00:00
---

Wait for the grep command to complete.

- Default model identifiers in shared/models.ts (DEFAULT_TEXT_MODEL,
DEFAULT_IMAGE_MODEL, DEFAULT_AUDIO_MODEL, DEFAULT_VIDEO_MODEL,
DEFAULT_MUSIC_MODEL) strip the 'models/' prefix and explicitly exclude 'exp',
'experimental', and 'latest' aliases in favor of stable or preview releases.

- Default model identifiers are configured in `shared/models.ts`
(`DEFAULT_TEXT_MODEL`, `DEFAULT_IMAGE_MODEL`, `DEFAULT_AUDIO_MODEL`,
`DEFAULT_VIDEO_MODEL`, `DEFAULT_MUSIC_MODEL`), stripping the `models/` prefix
and excluding experimental or `latest` tags.

- Default Gemini and Lyria model IDs are centralized in shared/models.ts
(DEFAULT_TEXT_MODEL, DEFAULT_IMAGE_MODEL, DEFAULT_AUDIO_MODEL,
DEFAULT_VIDEO_MODEL, and DEFAULT_MUSIC_MODEL).

- Default model IDs are defined in `shared/models.ts` (`DEFAULT_TEXT_MODEL`,
`DEFAULT_IMAGE_MODEL`, `DEFAULT_AUDIO_MODEL`, `DEFAULT_VIDEO_MODEL`,
`DEFAULT_MUSIC_MODEL`) without the `models/` prefix.
- Default model selection criteria restrict models to Gemini variants (or Lyria
for music) while excluding `exp`, `experimental`, and `latest` aliases.

- Default Gemini and Lyria models are centralized in `shared/models.ts`
(`DEFAULT_TEXT_MODEL`, `DEFAULT_IMAGE_MODEL`, `DEFAULT_AUDIO_MODEL`,
`DEFAULT_VIDEO_MODEL`, and `DEFAULT_MUSIC_MODEL`) with the `models/` prefix
stripped.
