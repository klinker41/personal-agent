---
topic: crowdsec-parser-whitelists
category: knowledge
tags: [knowledge, crowdsec-parser-whitelists]
updated_at: 2026-11-25T07:16:48.637693+00:00
confidence: 0.95
---

# Knowledge: Crowdsec-Parser-Whitelists

- Immich mobile clients (immich-ios) can burst GET
/api/assets/<uuid>/thumbnail?size=thumbnail&edited=true requests that return 404
when edited thumbnails do not exist, triggering crowdsecurity/http-probing
false-positive bans.
- To exempt benign endpoint-specific 404s in CrowdSec without weakening
http-probing globally, add an s02-enrich whitelist parser
(/etc/crowdsec/parsers/s02-enrich/*.yaml) scoped by host and URI regex, restart
CrowdSec, and verify both matching and non-matching log lines with cscli
explain.

- Immich mobile clients (`immich-ios`) requesting bursts of missing asset
thumbnails (`GET /api/assets/.../thumbnail` returning HTTP 404) on
`photos.klinker-cabin.computer` can false-positive trigger
`crowdsecurity/http-probing` and issue a 4-hour WAN IP ban.

- Bursts of HTTP 404 responses on Jellyfin's GET /Audio/<id>/Lyrics endpoint
(such as from jellyfin-sdk-kotlin clients querying tracks without lyrics) can
trigger the crowdsecurity/http-probing scenario and issue false-positive bans
via cs-unifi-bouncer.
