# Open WebUI

Open WebUI provides a browser interface for the OpenAI-compatible API exposed
by [`llama-cpp`](../llama-cpp/README.md). In this repository it connects to
the `llama-server` container over the shared `webgateway` Docker network and is
published by Traefik at `https://webui.<DOMAIN>`.

## Requirements

- Docker Engine with Docker Compose
- The external Docker network `webgateway`
- A running `llama-cpp` stack reachable as `llama-server:8080`
- Traefik configured to route `webui.<DOMAIN>`

## Configuration

Copy the environment template:

```bash
cp .env.example .env
```

Set the following variables:

| Variable | Purpose |
| --- | --- |
| `DOMAIN` | Creates the Traefik host rule `webui.<DOMAIN>` |
| `OPENAI_API_KEY` | Value sent to the upstream OpenAI-compatible backend |

`llama-cpp` does not enforce authentication by default, but Open WebUI still
expects an API key field for the OpenAI provider. Any non-empty placeholder is
sufficient unless the upstream API is later protected.

This compose file sets `WEBUI_AUTH=false`, which disables the Open WebUI login
screen. Keep this service behind trusted network access or add auth at the
Traefik layer before exposing it more broadly.

## Start the service

Create the external network once if it does not already exist:

```bash
docker network create webgateway
```

Start the UI:

```bash
docker compose up -d
docker compose logs -f open-webui
```

Check the rendered Compose configuration with:

```bash
docker compose config
```

## Access the UI

Open:

```text
https://webui.example.com
```

Replace `example.com` with the configured domain.
