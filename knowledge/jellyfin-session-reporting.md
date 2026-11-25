---
topic: jellyfin-session-reporting
category: knowledge
tags: [knowledge, jellyfin-session-reporting]
updated_at: 2026-11-25T07:07:25.743710+00:00
confidence: 0.95
---

# Knowledge: Jellyfin-Session-Reporting

- On Jellyfin Server, Audio items in Music libraries have SupportsPlayedStatus =
true and SupportsPositionTicksResume = false; calling SessionApi endpoints
(/Sessions/Playing, /Sessions/Playing/Progress, /Sessions/Playing/Stopped) for
Audio items causes OnPlaybackStart to immediately mark Played = true and
OnPlaybackProgress/OnPlaybackStop to reset PlaybackPositionTicks = 0 while
broadcasting UserDataChanged WebSocket events.
- To preserve resume positions and unplayed status for long-form audio stored as
Jellyfin Audio items (such as podcasts), clients must skip SessionApi playback
reporting and instead sync playbackPositionTicks and played state directly via
UserDataApi.

- When combining live Jellyfin progress reporting (`reportPlaybackProgress`,
`reportPlaybackStop`, or `/UserItems/{itemId}/UserData`) with an offline sync
queue, every successful online progress/stop update must update local state and
clear any pending dirty (`toBeSynced`) flag; otherwise an early transient
network drop leaves a stale queued position that later background syncs push
over newer server progress.
- Playback reporters should re-send `reportPlaybackStart` when coming online
after an offline start or resuming from a pause >= 15 minutes to re-establish
expired Jellyfin playback sessions.
