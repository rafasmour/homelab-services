# llama.cpp model router

Dockerized [`llama-server`](https://github.com/ggml-org/llama.cpp) configured as
an OpenAI-compatible model router. Models are selected by the alias defined in
`models/router-config.ini` and loaded from the local `models/` directory.

The image builds llama.cpp with Vulkan support and is intended for an AMD GPU
available through `/dev/dri`. Traefik publishes the API at
`https://ai.<DOMAIN>`; port 8080 is not published directly on the host.

## Requirements

- Docker Engine with Docker Compose
- A working AMDGPU/Mesa Vulkan setup on the host
- Access to `/dev/dri` for Docker containers
- An existing external Docker network named `webgateway`, used by Traefik
- One or more local GGUF model files

## Configuration

Copy the environment template:

```bash
cp .env.example .env
```

Set `DOMAIN` to the base domain handled by Traefik. The remaining variables tune
the build and the router-wide llama.cpp defaults:

| Variable | Purpose |
| --- | --- |
| `DOMAIN` | Creates the Traefik host rule `ai.<DOMAIN>` |
| `CONTEXT_SIZE` | Default context size for presets that do not override it |
| `BATCH_SIZE` | Logical prompt-processing batch size |
| `UBATCH_SIZE` | Physical prompt-processing batch size |
| `MAX_JOBS` | Parallel jobs used while compiling llama.cpp |
| `LLAMA_REF` | llama.cpp tag or commit to build; pin this for reproducibility |

`MODEL_FILE` is not used by this router setup. Models are configured through
the preset file instead of a single `-m` or `-hf` argument.

Copy the router configuration template and add the required GGUF files:

```bash
cp models/router-config.ini.example models/router-config.ini
```

Each INI section is a model alias accepted by the API. Paths are container
paths and must therefore start with `/models/`:

```ini
[chat-model]
model = /models/llama-3-8b.gguf
ctx-size = 8192
ngl = 99

[code-model]
model = /models/deepseek-coder.gguf
ctx-size = 16384
ngl = 99
```

Put `llama-3-8b.gguf` and `deepseek-coder.gguf` in `models/` for this example.
The container runs as a non-root user, so the directory and model files must be
readable by that user. GGUF files and the active `router-config.ini` are ignored
by Git; only the example configuration is committed.

Preset options override matching router-wide defaults from `docker-compose.yml`.
The current global defaults enable full GPU offload, Flash Attention, quantized
KV cache, Jinja chat templates, and the metrics endpoint. Tune context and batch
sizes to the available VRAM and the models in use.

## Start the router

Create the external network once if Traefik has not already created it:

```bash
docker network create webgateway
```

Build and start the service:

```bash
docker compose up --build -d
docker compose logs -f llama-cpp
```

Check the effective Compose configuration with:

```bash
docker compose config
```

## Use the API

List the configured models:

```bash
curl https://ai.example.com/v1/models
```

Select a preset by passing its section name in the OpenAI-compatible `model`
field:

```bash
curl https://ai.example.com/v1/chat/completions \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "chat-model",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```

Replace `example.com` and `chat-model` with the configured domain and preset
alias. The llama.cpp server does not add authentication in this repository;
configure access control in Traefik before exposing it outside a trusted
network.

## Update models or presets

After changing `models/router-config.ini` or replacing a model, restart the
service so the router reads the updated configuration:

```bash
docker compose restart llama-cpp
```

To build a different llama.cpp revision, update `LLAMA_REF` in `.env` and
rebuild:

```bash
docker compose build --no-cache llama-cpp
docker compose up -d
```
