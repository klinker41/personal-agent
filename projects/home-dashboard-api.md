---
topic: home-dashboard-api
category: project
tags: [project, home-dashboard-api]
updated_at: 2026-10-14T00:03:29.535621+00:00
confidence: 0.95
---

# Project: Home-Dashboard-Api

- Python API service (Docker image `jklinker/home-api-server:latest` , default
port 5002) aggregating metrics from Emporia Vue, Airthings sensors, Teslemetry,
and Unraid GraphQL endpoints.
- Configured entirely via environment variables (e.g., `VUE_USERNAME` ,
`AIRTHINGS_CLIENT_ID` , `TESLEMETRY_API_KEY` , `UNRAID_*_API_KEY` ) and deployed
via Docker Compose.

- Docker image `jklinker/home-api-server:latest` runs `api_server.py` exposed on
port 5002 using exec-form `CMD ["python", "-u", "api_server.py"]` .
