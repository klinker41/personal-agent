---
topic: jellyfin-api-v12
category: knowledge
tags: [knowledge, jellyfin-api-v12]
updated_at: 2026-09-12T00:35:11.214211+00:00
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
