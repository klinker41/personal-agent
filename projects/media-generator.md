---
topic: media-generator
category: project
tags: [project, media-generator]
updated_at: 2026-09-07T00:34:30.849657+00:00
confidence: 0.95
---

# Project: Media-Generator

- **Platform & Branding**: Branded as 'Media Studio' in the top app bar (not
  'AI Media Studio' or 'Podcast Generator'). Built as a unified Bun and Hono
  web service with JWT authentication, combining cinematic movie generation
  and episodic podcast synthesis.
- **Cinematic Movie Pipeline**: 7-stage workflow spanning prompting, plot
  formulation, screenplay breakdown, character casting with reference
  portraits, scene chunking with camera setups/plates, Gemini Omni video
  generation with temporal continuity, and FFmpeg stitching with single-chunk
  regeneration and upscaling.
- **Episodic Podcast Pipeline**: Integrates Gemini TTS multi-speaker dialogue
  synthesis with celestial voice profiles, ID3v2-tagged MP3 mastering,
  automated cron releases, Jellyfin/Emby media server triggers, and RSS 2.0
  feeds with iTunes tags.
- **Asset Storage**: Consolidated all podcast assets directly under
  `data/podcasts/` (`episodes`, `outputs`, `speaker_previews`), removing
  `OUTPUTS_DIR` and legacy directory fallback logic. Enforced
  `RESERVED_PODCAST_DIRS` in `fileStore.ts` to prevent static asset folders
  from colliding with podcast UUID directories during listing, lookup, and
  deletion.
- **Performance**: Replaced $O(N \times M)$ per-podcast disk scans in
  `PodcastStore.list()` with a single-pass active generation `Set`, and added
  a 5-second mutation-invalidated in-memory cache to `EpisodeStore`.
