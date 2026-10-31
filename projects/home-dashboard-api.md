---
topic: home-dashboard-api
category: project
tags: [project, home-dashboard-api]
updated_at: 2026-10-24T06:01:09.964414+00:00
confidence: 0.95
---

# Project: Home-Dashboard-Api

- Python API service (Docker image `jklinker/home-api-server:latest`, default
port 5002) aggregating metrics from Emporia Vue, Airthings sensors, Teslemetry,
and Unraid GraphQL endpoints.
- Configured entirely via environment variables (e.g., `VUE_USERNAME`,
`AIRTHINGS_CLIENT_ID`, `TESLEMETRY_API_KEY`, `UNRAID_*_API_KEY`) and deployed
via Docker Compose.

- Docker image `jklinker/home-api-server:latest` runs `api_server.py` exposed on
port 5002 using exec-form `CMD ["python", "-u", "api_server.py"]`.

- Runs as Docker container `home-api-server` on `unRAID NAS`, serving
`/api/energy`, `/api/air`, `/api/solar`, and `/api/unraid`.
- `get_unraid_notifications` in `api_server.py` catches `RequestException` from
Unraid GraphQL endpoints (such as `unraid-aibox.klinker-cabin.computer/graphql`)
and degrades gracefully by returning `0` for unreachable hosts while continuing
to serve `nas` and `seedbox` notification counts.
