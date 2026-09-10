---
topic: jellyfin-api-v12
category: knowledge
tags: [knowledge, jellyfin-api-v12]
updated_at: 2026-09-10T00:01:32.559974+00:00
confidence: 0.95
---

# Knowledge: Jellyfin-Api-V12

- Jellyfin 12 disabled the legacy `X-Emby-Token` authentication header by
default; requests must use `Authorization: MediaBrowser Token="<api-key>"`.
- The user-scoped metadata endpoint `GET /Users/{userId}/Items/{itemId}` was
removed in Jellyfin 12; use `GET /Items/{itemId}?userId={userId}` instead.
- Jellyfin 12 image upload (`POST /Items/{itemId}/Images/{imageType}`) requires
raw binary buffer data (`format: binary`) in the request body rather than
base64-encoded strings.
- `GET /Users/Me` can be used to directly resolve the authenticated user ID for
an API key holder instead of querying all users via `GET /Users`.
