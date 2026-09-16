# Ollama networking on Windows + WSL2

This document explains the networking setup required for LiteLLM (running in Podman/WSL2) to reach Ollama (running on the Windows host with GPU access).

## Problem summary

- Ollama runs natively on Windows to use the GPU.
- LiteLLM, Grafana LGTM, and other services run in Podman containers inside WSL2.
- By default, containers in WSL2 cannot reach `localhost` on the Windows host.
- `host.docker.internal` is unreliable in some Podman/WSL2 setups.

**Chosen approach:**  
Hard‑code the WSL2 gateway IP (the “WSL adapter” IP) as the Ollama address in `litellm_config.yaml`. This is simple, stable, and easy to document.

---

## How it works

- WSL2 uses a virtual network with NAT.
- The Windows host is reachable from WSL2 via the **default gateway IP** of the WSL2 network.
- From inside WSL2 (and containers running in WSL2), that IP is effectively “the Windows host”.

So LiteLLM must call Ollama at:

```text
http://<WSL2_GATEWAY_IP>:11434
```

---

## Obtaining the WSL2 gateway IP

From any WSL2 shell (Ubuntu, Debian, etc.):

```bash
# Print the WSL2 gateway IP (Windows host from WSL2 perspective)
WSL_GATEWAY_IP=$(ip route | grep default | awk '{print $3}')
echo "$WSL_GATEWAY_IP"
```

Example output:

```text
172.25.160.1
```

This IP may change if you restart WSL or change networking, so treat it as environment‑specific.

---

## Configuring LiteLLM

1. Get the gateway IP:

   ```bash
   WSL_GATEWAY_IP=$(ip route | grep default | awk '{print $3}')
   echo "$WSL_GATEWAY_IP"
   ```

2. Edit `litellm_config.yaml` and set `api_base` for the Ollama model:

   ```yaml
   model_list:
     - model_name: qwen3-local
       litellm_params:
         model: ollama/qwen3:14b
         api_base: http://<WSL_GATEWAY_IP>:11434
   ```

   Replace `<WSL_GATEWAY_IP>` with the value from step 1, e.g.:

   ```yaml
   model_list:
     - model_name: qwen3-local
       litellm_params:
         model: ollama/qwen3:14b
         api_base: http://172.25.160.1:11434
   ```

3. Restart LiteLLM:

   ```bash
   podman compose up -d --force-recreate litellm
   ```

---

## Verifying connectivity from the container

From WSL2:

```bash
WSL_GATEWAY_IP=$(ip route | grep default | awk '{print $3}')

podman exec -it litellm curl -s "http://$WSL_GATEWAY_IP:11434/api/tags"
```

Expected result: JSON with your Ollama models, e.g.:

```json
{
  "models": [
    {
      "name": "qwen3:14b",
      ...
    }
  ]
}
```

If you get this, LiteLLM can reach Ollama.

---

## Notes and alternatives

- This guide uses a **hard‑coded IP** in `litellm_config.yaml` for simplicity.
- If the WSL2 gateway IP changes (e.g. after WSL restart), update `api_base` accordingly.
- More advanced setups can:
  - Use an env var `OLLAMA_HOST_IP` and substitute it in the config.
  - Generate `litellm_config.yaml` at startup from a template.
- `host.docker.internal` was considered but proved unreliable in this environment.

For day‑to‑day work, the hard‑coded IP approach minimizes friction and lets you focus on building features and observability instead of fighting Windows virtualization quirks.