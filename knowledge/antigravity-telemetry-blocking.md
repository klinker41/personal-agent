---
topic: antigravity-telemetry-blocking
category: knowledge
tags: [knowledge, antigravity-telemetry-blocking]
updated_at: 2026-08-29T11:55:00.829319+00:00
confidence: 0.95
---

# Knowledge: Antigravity-Telemetry-Blocking

- Google separates Gemini inference endpoints (cloudaicompanion.googleapis.com,
cloudcode-pa.googleapis.com, generativelanguage.googleapis.com) and
authentication endpoints (accounts.google.com, oauth2.googleapis.com) from
telemetry endpoints.
- Telemetry and diagnostics domains (firebaselogging-pa.googleapis.com,
feedback-pa.googleapis.com, cloudtrace.googleapis.com,
clouderrorreporting.googleapis.com) can be safely mapped to 0.0.0.0 in
/etc/hosts without breaking AI completions or auth.
- Telemetry export can be suppressed via environment variables: DO_NOT_TRACK=1,
OTEL_SDK_DISABLED=true, OTEL_TRACES_EXPORTER=none, OTEL_METRICS_EXPORTER=none,
and OTEL_LOGS_EXPORTER=none.
