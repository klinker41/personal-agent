---
topic: exoplayer-playback-lifecycle
category: knowledge
tags: [knowledge, exoplayer-playback-lifecycle]
updated_at: 2026-11-27T07:46:05.320296+00:00
compacted_at: 2026-11-27T07:46:05.320296+00:00
confidence: 0.95
---

# Knowledge: Exoplayer-Playback-Lifecycle

## Lifecycle, Position & Audio Focus

- In AndroidX Media3 ExoPlayer, `clearMediaItems()` or item transitions
  reset `currentPosition` to `0L` and fire
  `Player.Listener.onPositionDiscontinuity` before `onMediaItemTransition`.
  Listeners syncing `currentPosition` on discontinuity will overwrite the
  outgoing item's final position unless captured beforehand or read from
  the outgoing `PositionInfo`.
- Lazily recreating a torn-down ExoPlayer via `ensurePlayerInitialized()`
  resets speed to `1.0x`; re-apply persisted playback speed after
  initialization on resume/play rather than relying on in-memory session
  state.
- In Media3, `AudioFocusManager.willPauseWhenDucked()` returns `true` for
  `C.AUDIO_CONTENT_TYPE_SPEECH`, setting `setWillPauseWhenDucked(true)` so
  `AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK` pauses instead of ducking; use
  `C.AUDIO_CONTENT_TYPE_MUSIC` or `C.AUDIO_CONTENT_TYPE_MOVIE` to duck.
  `AudioFocusManager.setAudioAttributes()` does not rebuild a cached
  `AudioFocusRequest` when only content type changes, so switching an
  active player from `AUDIO_CONTENT_TYPE_SPEECH` to
  `AUDIO_CONTENT_TYPE_MUSIC` retains `willPauseWhenDucked = true`.

## Queue Transitions & Advancement

- When manually jumping to a queued item before async media preparation
  completes, call `exoPlayer.stop()` right after reporting the outgoing
  item's position to prevent audio bleed and mid-transition
  `Player.STATE_ENDED` triggers.
- When advancing asynchronously across separate playback engines, defer
  tearing down or switching the active engine until incoming preparation
  succeeds, reset advancement state on `CancellationException` before
  rethrowing, and dispatch `ACTION_STOP` if all remaining queued items
  fail.
- When coordinating background service advancement guards (such as
  `isAdvancing`) with optional Compose navigation collectors that may be
  absent during background playback, replace `yield()` ordering with
  generation-tagged events acknowledged after navigation completes,
  falling back to immediate completion when there are no subscribers.

## UI & State Concurrency

- Relative seeks (e.g., 10s rewind/forward) briefly transition ExoPlayer
  into `STATE_BUFFERING`; derive center playback control visibility
  declaratively from `showControls && !isControlsLocked` rather than
  imperatively clearing button visibility on `STATE_BUFFERING`.
- When ViewModel flows launch concurrent `Dispatchers.IO` tasks (e.g.,
  segment loading) alongside main-thread player callbacks, mutate
  `MutableStateFlow` with atomic `.update { ... }` instead of non-atomic
  `_uiState.value = _uiState.value.copy(...)` to prevent lost-update races.

## Testing

- With a relaxed MockK `SimpleCache`, stub `getContentMetadata(any())` to
  return `DefaultContentMetadata.EMPTY`; otherwise content length defaults
  to `0L` instead of `C.LENGTH_UNSET` (`-1L`), causing
  `CacheDataSource.open()` to hit an immediate 0-byte EOF and emit async
  `ERROR_CODE_PARSING_CONTAINER_UNSUPPORTED` on the playback thread.
- In Robolectric tests (multi-item loads or paused `MainLooper`), async
  `PlaybackException`s from `ExoPlayer:Playback` (such as
  `ERROR_CODE_PARSING_CONTAINER_UNSUPPORTED` on 0-byte dummy files) sit
  queued on `Looper.getMainLooper()` until pumped and leak across shared
  `Player`/`ViewModel` instances; isolate each scenario in its own test
  harness and avoid 0-byte local files.
