---
topic: media-generator
category: project
tags: [project, media-generator]
updated_at: 2026-09-10T00:33:59.133203+00:00
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
  regeneration and upscaling. In `movieGemini.ts`, scene chunking enforces
  shot/reverse-shot continuity where speaker alternations mandate `new_shot`,
  making continuous camera shots rare outside extended single-speaker dialogue
  exceeding chunk limits.
- **Episodic Podcast Pipeline**:
  - Synthesizes multi-speaker dialogue via Gemini TTS with celestial voice
    profiles, ID3v2-tagged MP3 mastering, automated cron releases, and RSS 2.0
    feeds with iTunes tags.
  - In `formatSpeakerGuidelines`, one host must always announce the podcast
    name at the start regardless of banter setting; banter-off mode starts
    naturally and transitions directly into discussion without small talk.
  - Supports ad reads via the `ad_reads` property on `Podcast`, distributing
    sponsor segments evenly across generated episode dialogue.
  - Theme music uses Google Lyria 3 Clip (`lyria-3-clip-preview`, overridable
    via `MUSIC_MODEL` or `GEMINI_MUSIC_MODEL`). Audio files are stored in
    `data/podcasts/music/` (`music_{intro|outro}_<timestamp>_<uuid>.mp3`),
    tracked in `podcast.json` (`intro_music_path`, `outro_music_path`), and
    served with HTTP Range support at `/api/podcasts/music/:filename`.
  - Episode assembly adds a 15s intro music clip (2s fade-in, 3s fade-out) and
    a 30s outro music cl
<truncated 999 bytes>
ed), excluding
    Imagen/Veo. The `models/` prefix must be stripped.
  - Tier rules require Flash for text, audio, and video, and Pro for image
    generation. Variants like `exp`, `experimental`, and `latest` are
    strictly excluded, while preview models are permitted.
  - Automated updates: The model updater sidecar
    (`antigravity-plugin/model-updater`) targets `media-generator`
    (`projectId: 041f44fe-de8b-42ae-8716-67010ae98326`), committing changes
    locally and alerting via Slack webhook; pushing requires user approval.
- **Asset Storage & Environment**:
  - Podcast assets are consolidated under `data/podcasts/` (`episodes`,
    `outputs`, `speaker_previews`, `music`), eliminating `OUTPUTS_DIR` and
    legacy fallback logic.
  - `RESERVED_PODCAST_DIRS` in `fileStore.ts` prevents static asset folders
    from colliding with podcast UUID directories during listing, lookup, and
    deletion.
  - `EXTERNAL_OUTPUTS_DIR` defaults to `/app/data/outputs-external` in Docker
    and an empty string (disabled) in backend file store if unset.
- **Performance Optimizations**:
  - Replaced $O(N \times M)$ per-podcast disk scans in `PodcastStore.list()`
    with a single-pass active generation `Set`.
  - Added a 5-second mutation-invalidated in-memory cache to `EpisodeStore`.
- **UI & Frontend Layout**:
  - Dashboard: Desktop (`lg:`) uses an asymmetric 2-column layout (65%
    creations feed, 35% operations sidebar); mobile (`<sm`) collapses to a
    single column with a 2x2 telemetry grid. Includes KPI cards
    (`DashboardCards.tsx`), an in-flight pipeline banner
    (`InFlightPipelineBanner.tsx`) with visualizer links, and a creations feed
    (`MediaCreationsFeed.tsx`). Documentation screenshots are stored in
    `docs/images/` (`dashboard-desktop.jpg`, `dashboard-mobile.jpg`).
  - Podcasts List: Mobile layout replaces nested container padding
    (`max-w-7xl px-4 py-8`) with `space-y-6 w-full`, using responsive cards
    with top-right status toggles, metadata badges, and expanded bottom action
    footers.
