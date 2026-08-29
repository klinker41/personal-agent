---
topic: ollama-inbound-auth
category: knowledge
tags: [knowledge, ollama-inbound-auth]
updated_at: 2026-08-29T11:54:21.290086+00:00
confidence: 0.95
---

# Knowledge: Ollama-Inbound-Auth

- The official Ollama container does not natively authenticate incoming HTTP
requests (`OLLAMA_API_KEY` in environment is only used for outbound
registry/cloud requests).
- To secure an Ollama host behind Nginx Proxy Manager, Bearer token
authentication can be enforced in advanced settings via `if ($http_authorization
!= "Bearer <TOKEN>") { return 401 '...'; }` with `proxy_buffering off;
proxy_cache off; proxy_read_timeout 600s;` to support long streaming responses.
