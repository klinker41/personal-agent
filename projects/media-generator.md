---
topic: media-generator
category: project
tags: [project, media-generator]
updated_at: 2026-10-14T00:38:07.801778+00:00
---

# Project: Media-Generator

## Model Configuration & Selection
- **Centralized Model IDs:** Default constants across modalities are defined
  in `shared/models.ts` and mirrored in `.env.example`, stripped of any
  `models/` prefix:
  - `DEFAULT_TEXT_MODEL`: `gemini-3.8-flash` (text)
  - `DEFAULT_IMAGE_MODEL`: `gemini-3-pro-image` (image)
  - `DEFAULT_AUDIO_MODEL`: `gemini-3.8-flash-tts` (audio)
  - `DEFAULT_VIDEO_MODEL`: `gemini-omni-1.1-flash` (video)
  - `DEFAULT_MUSIC_MODEL`: `lyria-3-clip-preview` (music)
- **Selection Criteria:** Project defaults restrict models to `gemini-*` (or
  `lyria-*` for music) and permit preview models, while strictly excluding
  experimental (`exp`, `experimental`) and alias (`latest`) models.

## Frontend Architecture & Authentication
- **Routing & Setup:** Authentication redirects to `/` instead of a dedicated
  `/login` route, managing first-time admin setup completion via `App.tsx`
  state.
- **Auth Provider:** A single `AuthProvider` wraps `App.tsx` internally rather
  than duplicating it in `main.tsx`.

## Backend Task Queues & Concurrency
- **Unified Queue Processing:** `JobQueue`, `AudiobookQueue`, and `BookQueue`
  inherit from `BaseTaskQueue` with re-entrancy guards to enforce FIFO gating
  and prevent concurrent execution or premature task completion.

## Storage Safety & Crash Recovery
- **Path Traversal & Safe Writes:** File stores (`EpisodeStore`, `UserStore`)
  and audiobook routes validate ID path segments and enforce directory
  containment via `dirPath.startsWith(OUTPUTS_DIR + path.sep)`, using atomic
  writes to prevent race conditions and unintended deletions.
- **Startup Crash Recovery:** Server startup executes
  `BookStore.recoverStuckM4bBuilds()` to automatically reconcile and resume
  interrupted audiobook compilation tasks stuck in `m4b_status: building`.
