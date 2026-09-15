# AI Harness Lab

A local lab for building an AI gateway with observability.

## Goal

Build this flow incrementally:

```text
Client / future agent
  -> LiteLLM proxy
  -> LLM provider

LiteLLM
  -> OpenTelemetry
  -> Grafana OTEL-LGTM
```

The current focus is the platform foundation. A local Ollama model and a CLI agent will be added later.

## Current state

- [x] Podman environment works and published container ports are reachable from Windows.
- [x] LiteLLM is running in a container.
- [x] LiteLLM configuration is mounted from `./litellm/config.yaml`.
- [x] PostgreSQL is running and connected to LiteLLM.
- [x] LiteLLM Admin UI is reachable and login works.
- [x] Grafana OTEL-LGTM is running.
- [x] Grafana UI is reachable and initial login/password change is complete.
- [ ] Configure a model in LiteLLM.
- [ ] Send a chat request through LiteLLM.
- [ ] Configure LiteLLM to export OpenTelemetry data.
- [ ] Verify LiteLLM traces, metrics, and OTel log records in Grafana.
- [ ] Add a local Ollama model.
- [ ] Add a simple swappable local CLI agent.
- [ ] Add a dedicated OTel Collector for processing, governance, and backend-independent routing.

## Components

| Component | Purpose |
|---|---|
| Podman | Runs the local containers. Docker Compose-compatible commands are used through Podman. |
| LiteLLM | The future AI gateway/proxy. Clients and agents will call LiteLLM rather than calling LLM providers directly. |
| PostgreSQL | Stores LiteLLM state needed by the Admin UI and future gateway features such as virtual keys, usage, budgets, and UI-managed configuration. |
| Grafana OTEL-LGTM | A single local container that bundles an OpenTelemetry Collector, Grafana UI, Tempo for traces, Loki for logs, and metrics storage. It is a lightweight lab backend, not the intended final production architecture. |

## Access

| Tool | URL | Login |
|---|---|---|
| LiteLLM Admin UI | `http://127.0.0.1:4000/ui` | Username: `admin`<br>Password: `LITELLM_MASTER_KEY` from `.env` |
| LiteLLM API | `http://127.0.0.1:4000/v1` | Future clients use a LiteLLM API key / configured authentication. |
| Grafana UI | `http://127.0.0.1:3000` | Username: `admin`<br>Password: `admin` then the password selected during Grafana's first-login password change. |

## Configuration

- `docker-compose.yaml` defines the running containers and their published ports.
- `.env` contains local secrets such as `LITELLM_MASTER_KEY` and the PostgreSQL password. Do not commit it.
- `.env.example` is the safe template committed to the repository.
- `litellm/config.yaml` is LiteLLM configuration mounted into the container as read-only.
- `model_list` is currently empty by design because a provider/model has not yet been added.

## Common commands

Run from the project folder containing `docker-compose.yaml`.

```powershell
# Start all services
podman compose up -d

# Show service status
podman compose ps

# Follow all logs
podman compose logs -f

# Follow one service's logs
podman logs -f litellm
podman logs -f otel-lgtm

# Stop services while keeping persistent data
podman compose down

# Stop services and delete local database/telemetry data
podman compose down -v
```

## Next checkpoint

Configure one model behind LiteLLM and make a successful chat request through the proxy. After that, connect LiteLLM's OpenTelemetry export to OTEL-LGTM and verify the request appears in Grafana.
