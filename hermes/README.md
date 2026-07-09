# Hermes

Docker Compose deployment for [Hermes Agent](https://github.com/NousResearch/hermes-agent) with:

- a dashboard exposed at `https://hermes.<DOMAIN>`
- an OpenAI-compatible API exposed at `https://hermes.<DOMAIN>/v1`
- a local `llama.cpp` backend reached over the shared `webgateway` Docker network

This README was checked against upstream Hermes documentation on 2026-07-09.

## Requirements

- Docker Engine with Docker Compose
- An existing external Docker network named `webgateway`
- A running `llama.cpp` service on that same network, reachable as `llama-cpp:8080`
- Traefik configured to route `hermes.<DOMAIN>`

## Configuration

Copy the template if you have not created a local env file yet:

```bash
cp .env.example .env
```

Set the following values in `.env`:

| Variable | Purpose |
| --- | --- |
| `DOMAIN` | Base domain used by Traefik. This stack serves `hermes.<DOMAIN>`. |
| `API_SERVER_KEY` | Bearer token for the Hermes OpenAI-compatible API. |
| `API_SERVER_MODEL_NAME` | Model name returned by the Hermes API, default `hermes-agent`. |
| `HERMES_DASHBOARD_BASIC_AUTH_USERNAME` | Dashboard basic auth username. |
| `HERMES_DASHBOARD_BASIC_AUTH_PASSWORD` | Dashboard basic auth password. Use a strong value. |
| `HERMES_DASHBOARD_BASIC_AUTH_SECRET` | Session secret for dashboard auth. Generate it with `openssl rand -hex 32`. |

Generate the dashboard auth secret with:

```bash
openssl rand -hex 32
```

Paste the resulting 64-character hex string into `HERMES_DASHBOARD_BASIC_AUTH_SECRET`.

## Current compose behavior

The Compose file is currently wired like this:

- `dashboard` runs `nousresearch/hermes-agent:latest` with the `dashboard` command
- `api` runs `nousresearch/hermes-agent:latest` with `gateway run`
- both services use `HERMES_PROVIDER=openai`
- Hermes sends model traffic to `http://llama-cpp:8080/v1`
- the dashboard talks to the internal API at `http://hermes-api:8642`

That means your `llama.cpp` container must be on the same Docker network and resolvable from Hermes.

## Start Hermes

Create the shared network once if it does not already exist:

```bash
docker network create webgateway
```

Start the stack:

```bash
docker compose up -d
docker compose logs -f
```

Inspect the resolved Compose configuration:

```bash
docker compose config
```

## Endpoints

- Dashboard: `https://hermes.<DOMAIN>`
- API base URL: `https://hermes.<DOMAIN>/v1`

Example API request:

```bash
curl https://hermes.example.com/v1/models \
  -H 'Authorization: Bearer replace-with-random-api-key'
```

Replace `example.com` and the bearer token with your actual values.

## Documentation used

- Hermes Agent upstream repository:
  `https://github.com/NousResearch/hermes-agent`
- Hermes README:
  `https://raw.githubusercontent.com/NousResearch/hermes-agent/main/README.md`
- Hermes environment variable reference:
  `https://hermes-agent.nousresearch.com/docs/reference/environment-variables`

Those sources were used to confirm the current Hermes install/docs location and the documented meaning of `OPENAI_BASE_URL`, `OPENAI_API_KEY`, and related environment-variable usage.
