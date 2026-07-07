# Homelab Services

This repository contains the containerized services I run in my homelab.
Each service directory includes its own setup notes and local configuration
templates.

## Services

| Service | Purpose | Setup |
| --- | --- | --- |
| [`homelab-ai`](./homelab-ai/README.md) | `llama.cpp` model router exposed through Traefik as an OpenAI-compatible API | Follow the [`homelab-ai` README](./homelab-ai/README.md) |
| [`openwebui`](./openwebui/README.md) | Web UI for the `homelab-ai` API, exposed through Traefik | Follow the [`openwebui` README](./openwebui/README.md) |

## Expected layout

Each service directory is self-contained and should include:

- `docker-compose.yml` with the service definition
- `README.md` with setup instructions
- `.env.example` with the required environment variables

Local-only files such as `.env`, downloaded models, caches, and runtime data
should remain untracked.
