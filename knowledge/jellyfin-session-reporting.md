---
topic: jellyfin-session-reporting
category: knowledge
tags: [knowledge, jellyfin-session-reporting]
updated_at: 2026-11-24T07:11:58.981992+00:00
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
