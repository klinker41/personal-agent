---
topic: exoplayer-playback-lifecycle
category: knowledge
tags: [knowledge, exoplayer-playback-lifecycle]
updated_at: 2026-11-25T07:05:06.264695+00:00
confidence: 0.95
---

# Knowledge: Exoplayer-Playback-Lifecycle

- In AndroidX Media3 ExoPlayer, calling clearMediaItems() or transitioning
between media items resets currentPosition to 0L and fires
Player.Listener.onPositionDiscontinuity before onMediaItemTransition; state
listeners that sync currentPosition on discontinuity will overwrite the outgoing
item's final position unless it is captured beforehand or read from the outgoing
PositionInfo.

- When an Android audio service tears down ExoPlayer and lazily recreates it via
ensurePlayerInitialized(), persisted playback speed must be re-applied after
initialization during resume/play rather than relying on in-memory session
state, because the new ExoPlayer instance resets to 1.0x.

- When mocking ExoPlayer's SimpleCache with a relaxed MockK mock, stub
getContentMetadata(any()) to return DefaultContentMetadata.EMPTY; otherwise
content length defaults to 0L instead of C.LENGTH_UNSET (-1L), causing
CacheDataSource.open() to hit an immediate 0-byte EOF and emit async
ERROR_CODE_PARSING_CONTAINER_UNSUPPORTED errors on the playback thread.
- In Robolectric tests that load multiple media items, isolate Player/ViewModel
instances per scenario or avoid 0-byte local files so delayed background
ExoPlayer PlaybackExceptions delivered on Looper.getMainLooper() cannot leak
into subsequent test phases.

- In Robolectric tests using ExoPlayer with a paused MainLooper, asynchronous
errors from the background ExoPlayer:Playback thread (such as
ERROR_CODE_PARSING_CONTAINER_UNSUPPORTED on 0-byte dummy files) sit queued on
MainLooper until pumped and can leak into subsequent loads on the same
player/ViewModel instance; isolate each playback scenario in its own test
harness.
- When ViewModel playback flows launch concurrent Dispatchers.IO tasks (e.g.,
segment loading) alongside main-thread player callbacks, always mutate
MutableStateFlow with atomic .update { ... } rather than non-atomic
read-modify-write assignments (_uiState.value = _uiState.value.copy(...)) to
prevent lost-update races.

- In AndroidX Media3, AudioFocusManager.willPauseWhenDucked() returns true for
C.AUDIO_CONTENT_TYPE_SPEECH, causing ExoPlayer to configure
setWillPauseWhenDucked(true) and pause instead of ducking on
AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK; use C.AUDIO_CONTENT_TYPE_MUSIC or
C.AUDIO_CONTENT_TYPE_MOVIE when ducking is desired.
- Media3's AudioFocusManager.setAudioAttributes() does not rebuild a cached
AudioFocusRequest when only content type changes, so switching an active player
from AUDIO_CONTENT_TYPE_SPEECH to AUDIO_CONTENT_TYPE_MUSIC retains
willPauseWhenDucked = true.
