---
topic: nginx-proxy-manager
category: knowledge
tags: [knowledge, nginx-proxy-manager]
updated_at: 2026-11-06T07:01:25.011286+00:00
confidence: 0.95
---

# Knowledge: Nginx-Proxy-Manager

- When the Nginx Proxy Manager (NPM) Proxy Hosts page renders a blank screen
while other pages work normally, it is typically caused by malformed JSON in
`/data/database.sqlite` within the `proxy_host` table (`domain_names`,
`locations`, or `meta` columns, such as unquoted keys or strings) or orphaned
`certificate_id`/`access_list_id` references that break `GET
/api/nginx/proxy-hosts?expand=owner,access_list,certificate`.
