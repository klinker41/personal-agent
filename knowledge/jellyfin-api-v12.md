---
topic: jellyfin-api-v12
category: knowledge
tags: [knowledge, jellyfin-api-v12]
updated_at: 2026-11-24T07:10:17.990056+00:00
confidence: 0.95
---

# Knowledge: Jellyfin-Api-V12

- Jellyfin 12 disabled the legacy `X-Emby-Token` authentication header by
default; requests must use `Authorization: MediaBrowser Token="<api-key>"`.

- Jellyfin API keys are server-level credentials without an associated session
user; calling `GET /Users/Me` fails with HTTP 400. To resolve a valid user ID,
query the admin user list via `GET /Users`.

- The user-scoped metadata endpoint `GET /Users/{userId}/Items/{itemId}` was
removed in Jellyfin 12; use `GET /Items/{itemId}?userId={userId}` instead.

- While general endpoints like `GET /Items` allow search and operations without
specifying a user, `GET /Items/{itemId}` strictly requires a `userId` query
parameter even with a server API key, returning HTTP 400 if omitted.

- Updating item metadata using an API key requires retrieving a user ID from
`GET /Users`, requesting item metadata via
`GET /Items/{itemId}?userId={userId}`, applying changes, and saving via
`POST /Items/{itemId}`.

- Image uploads (`POST /Items/{itemId}/Images/{imageType}`) require raw binary
buffer data (`format: binary`) in the request body rather than base64 strings.

- In Jellyfin audio item metadata, disc numbers map to 'ParentIndexNumber' and
track numbers map to 'IndexNumber'.

- Jellyfin's /Items/Resume endpoint filters out libraries listed in the user's
LatestItemsExcludes preference (commonly excluding Audiobooks and Podcasts);
query /Items directly with Recursive=true and Filters=IsResumable (plus
IncludeItemTypes such as AudioBook or Audio) to reliably fetch resumable items
from excluded libraries.

- Jellyfin persists per-user podcast episode playback state (Played, PlayCount,
LastPlayedDate, PlaybackPositionTicks) on Audio items in server-side UserData,
and marking a parent podcast album watched marks its child episodes played.
- Querying unplayed podcast episodes with isPlayed = false skips played recent
episodes and falls back to older unplayed backlog episodes from earlier in the
feed unless historical episodes are marked played.
