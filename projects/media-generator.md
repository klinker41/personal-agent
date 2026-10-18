---
topic: media-generator
category: project
tags: [project, media-generator]
updated_at: 2026-10-18T06:41:44.651134+00:00
---

# Project: Media-Generator

## Model Configuration & Selection
- **Centralized Model IDs & Selection:** Default generative model IDs across
  all media categories (text, image, audio, video, and music) are centralized
  as exported constants in `shared/models.ts` and `.env.example` without
  `models/` prefixes: `DEFAULT_TEXT_MODEL` (`gemini-3.8-flash`),
  `DEFAULT_IMAGE_MODEL` (`gemini-3-pro-image`), `DEFAULT_AUDIO_MODEL`
  (`gemini-3.8-flash-tts`), `DEFAULT_VIDEO_MODEL` (`gemini-omni-1.1-flash`),
  and `DEFAULT_MUSIC_MODEL` (`lyria-3-clip-preview`). Selection allows preview
  models within `gemini-*` or `lyria-*`, strictly excluding experimental
  (`exp`, `experimental`) and alias (`latest`) models.

## Architecture & Concurrency
- **Auth & Setup Routing:** Authentication redirects to `/` instead of a
  dedicated `/login` route, managing first-time admin setup completion via
  `App.tsx` state. A single `AuthProvider` wraps `App.tsx` internally rather
  than duplicating it in `main.tsx`.
- **Task Queues & Concurrency:** `JobQueue`, `AudiobookQueue`, and `BookQueue`
  inherit from `BaseTaskQueue` with re-entrancy guards to enforce FIFO gating,
  preventing concurrent task execution or premature completion.

## Storage Safety & Crash Recovery
- **Path Traversal & Safe Writes:** File stores (`EpisodeStore`, `UserStore`)
  and audiobook routes validate ID path segments and enforce directory
  containment via `dirPath.startsWith(OUTPUTS_DIR + path.sep)`, using atomic
  writes to prevent race conditions and unintended deletions.
- **Startup Crash Recovery:** Server startup executes
  `BookStore.recoverStuckM4bBuilds()` to automatically reconcile and resume
  interrupted audiobook compilation tasks stuck in `m4b_status: building`.
