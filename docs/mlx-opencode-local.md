# MLX OpenAI server + OpenCode (local)

This note describes **where each piece lives**, what the **“bridge”** is, how to **restart** on this Mac, and how to **reproduce the setup on another machine**.

---

## What runs where

| Piece | Role | Location on disk |
|--------|------|------------------|
| **mlx-openai-server** | OpenAI-compatible HTTP API (`/v1/chat/completions`, `/v1/models`, …) backed by MLX on Apple Silicon | Install dir: `~/mlx-openai-server/` (venv: `~/mlx-openai-server/.venv/`) |
| **Multi-model config** | Declares model paths, `served_model_name`, `on_demand`, port | `~/mlx-openai-server/models.yaml` |
| **Weights** | Local MLX model folders | Example: `~/MLX_Models/…` (your paths in `models.yaml`) |
| **OpenCode** | IDE / CLI client | App as installed; **config** in `~/.config/opencode/opencode.json` |
| **OpenCode auth** (local key placeholder) | Provider credentials file | `~/.local/share/opencode/auth.json` |

**Listen address:** the server binds **`0.0.0.0:8000`** (see `models.yaml` → `server.port`). Clients on the same machine use **`http://127.0.0.1:8000/v1`**.

---

## What “the bridge” means (not `mlx_lm.server`)

OpenCode does **not** talk to MLX directly. It uses the **OpenAI-compatible HTTP client** from the config field:

- `"npm": "@ai-sdk/openai-compatible"`
- `"options.baseURL": "http://127.0.0.1:8000/v1"`

So the chain is:

```text
OpenCode  →  OpenAI-compatible HTTP (`/v1/...`)  →  mlx-openai-server  →  MLX / Metal
```

That HTTP layer is what people loosely call the **bridge**: same wire shape as the OpenAI API, implemented locally by **mlx-openai-server**. This is **separate** from **`python -m mlx_lm.server`**, which is a different program; if that path broke for OpenCode, pointing **baseURL** at **mlx-openai-server** is the supported workaround.

---

## Long streamed replies and queue_timeout

**Default:** each model in `models.yaml` uses **`queue_timeout: 300`** unless you set it — that is **300 seconds = 5 minutes**. The server waits on an internal queue for the **next token chunk** from the worker; if that wait exceeds `queue_timeout`, you get **`TimeoutError`** (often with `CancelledError` underneath) and logs like **`Error in stream wrapper`** in `handle_stream_response`. A long OpenCode answer can hit that limit even though the model is still fine.

**What raising `queue_timeout` does:** the value is a **ceiling** (maximum seconds without a new chunk before the server aborts). It is **not** “you sit for two hours.” Streaming still arrives at normal speed; you are only allowing **longer total runs** before the server gives up. Pick any seconds you like, for example **`1800`** (30 min), **`3600`** (1 h), or **`7200`** (2 h). Anything **above `300`** fixes the common “died after a few minutes” case.

**Per-model YAML** (repeat under each entry in `~/mlx-openai-server/models.yaml`):

```yaml
    queue_timeout: 7200
```

**CLI (single-model launch):** `mlx-openai-server launch` accepts **`--queue-timeout`** (same meaning, seconds).

After editing **`models.yaml`**, **restart** the server (`Ctrl+C`, then `launch` again) so the new value loads.

---

## Multiple OpenCode windows (read this before “testing batching”)

**mlx-openai-server** can run **more than one HTTP request** against the same loaded model; inside the handler subprocess, work may be scheduled so a **continuous batcher** can see multiple in-flight generations. That is **not** the same as “run two full **OpenCode** apps and expect it to always feel like one cloud API.”

With **two OpenCode instances** talking to the **same** server and model you can still see:

- **Cancelled HTTP streams** when one UI stops, retries, or navigates away → the server’s streaming task is torn down; you may see `asyncio.CancelledError` in logs.
- **`TimeoutError` on `result_queue.get()`** when the parent waits too long for the **next chunk** from the child (cancel, stall, overload, or the default **5 minute** `queue_timeout`). Long single replies: see **[Long streamed replies and queue_timeout](#long-streamed-replies-and-queue_timeout)** above.

So **MLX batching ≠ “two OpenCode clients are a supported stress test.”** For day-to-day stability on one Mac:

- Use **one OpenCode** session (or one active chat) against **one** `mlx-openai-server` on **one** port.
- If you already hit a bad state: **Ctrl+C** the server, wait for clean exit, **`launch` again**, restart OpenCode.

If you truly need two separate operator sessions, the boring reliable options are **two machines**, or **two servers on different ports** each with its **own** loaded model copy (memory-heavy and usually not worth it on a laptop).

---

## Restart on **this** machine (copy-paste)

**1. Stop the old server** (in the terminal where it is running): `Ctrl+C` and wait until the prompt returns. Do **not** start a second `launch` on the same port while a child process is still exiting.

**2. Start the server again:**

```bash
cd ~/mlx-openai-server
source .venv/bin/activate
mlx-openai-server launch --config ~/mlx-openai-server/models.yaml
```

Leave this terminal open. Wait for `Uvicorn running on http://0.0.0.0:8000`.

**3. Optional quick check** (second terminal):

```bash
curl -sS http://127.0.0.1:8000/v1/models | head -c 1500
```

**4. OpenCode** (after any edit to `~/.config/opencode/opencode.json`): quit OpenCode completely and reopen it, then pick e.g. `mlx/glm-5.1-mxfp4-q8`.

**5. Project / session** (if you use OpenCode from a repo):

```bash
cd /Users/jonathanrothberg/Rothberg-Terminal
opencode
```

---

## Full setup on **another** Apple Silicon Mac

Prerequisites: **macOS**, **Metal** (real machine, not a sandbox without GPU), **Homebrew**, local copies of the same model directories (or different paths you edit in YAML).

### A. Install `uv` and `mlx-openai-server`

```bash
if ! command -v uv >/dev/null 2>&1; then
  brew install uv
fi

mkdir -p "$HOME/mlx-openai-server"
cd "$HOME/mlx-openai-server"

uv python install 3.12
uv venv --python 3.12
source .venv/bin/activate
uv pip install --upgrade pip
uv pip install mlx-openai-server
```

### B. Write `models.yaml` on that machine

Replace **`YOURNAME`** and every **`model_path`** with paths that exist on **that** Mac. Keep `served_model_name` values in sync with OpenCode (next section).

```yaml
# $HOME/mlx-openai-server/models.yaml
server:
  host: 0.0.0.0
  port: 8000
  log_level: INFO

models:
  - model_path: /Users/YOURNAME/MLX_Models/GLM-5.1-MXFP4-Q8
    model_type: lm
    served_model_name: glm-5.1-mxfp4-q8
    on_demand: true
    on_demand_idle_timeout: 300
    queue_timeout: 7200
  - model_path: /Users/YOURNAME/MLX_Models/MiniMax-M2.7-8bit-gs32
    model_type: lm
    served_model_name: minimax-m2.7-8bit-gs32
    on_demand: true
    on_demand_idle_timeout: 300
    queue_timeout: 7200
  - model_path: /Users/YOURNAME/MLX_Models/DeepSeek-V4-Flash-mxfp8
    model_type: lm
    served_model_name: deepseek-v4-flash-mxfp8
    on_demand: true
    on_demand_idle_timeout: 300
    queue_timeout: 7200
  - model_path: /Users/YOURNAME/MLX_Models/deepseek-ai-DeepSeek-V4-Flash-8bit
    model_type: lm
    served_model_name: deepseek-v4-flash-8bit
    on_demand: true
    on_demand_idle_timeout: 300
    queue_timeout: 7200
```

Start the server on that machine:

```bash
cd ~/mlx-openai-server
source .venv/bin/activate
mlx-openai-server launch --config ~/mlx-openai-server/models.yaml
```

### C. OpenCode config on that machine

- If OpenCode runs **on the same Mac** as the server, use **`http://127.0.0.1:8000/v1`**.
- If OpenCode runs **on another computer** and reaches this Mac over the LAN, set **`baseURL`** to `http://<SERVER_LAN_IP>:8000/v1` and ensure **firewall** allows port **8000** (only on trusted networks).

Create/update **`~/.config/opencode/opencode.json`**:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "model": "mlx/glm-5.1-mxfp4-q8",
  "small_model": "mlx/glm-5.1-mxfp4-q8",
  "provider": {
    "mlx": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "MLX (local)",
      "options": {
        "baseURL": "http://127.0.0.1:8000/v1",
        "apiKey": "not-needed",
        "timeout": 900000,
        "chunkTimeout": 1200000
      },
      "models": {
        "glm-5.1-mxfp4-q8": {
          "name": "GLM 5.1 MXFP4 Q8",
          "tool_call": false,
          "reasoning": false,
          "limit": { "context": 131072, "output": 16000 }
        },
        "minimax-m2.7-8bit-gs32": {
          "name": "MiniMax M2.7 8bit gs32",
          "tool_call": true,
          "limit": { "context": 131072, "output": 16000 }
        },
        "deepseek-v4-flash-mxfp8": {
          "name": "DeepSeek V4 Flash mxfp8",
          "tool_call": false,
          "reasoning": true,
          "limit": { "context": 131072, "output": 16000 }
        },
        "deepseek-v4-flash-8bit": {
          "name": "DeepSeek V4 Flash 8bit",
          "tool_call": false,
          "reasoning": true,
          "limit": { "context": 131072, "output": 16000 }
        }
      }
    }
  }
}
```

Create **`~/.local/share/opencode/auth.json`**:

```json
{
  "mlx": {
    "type": "api",
    "key": "not-needed"
  }
}
```

Restart OpenCode after writing these files.

### D. Rules that avoid pain

- **`queue_timeout`:** default **300** (5 minutes) can kill long streams; see **[Long streamed replies and queue_timeout](#long-streamed-replies-and-queue_timeout)**.
- **`served_model_name` in YAML** must match the **keys** under `provider.mlx.models` in OpenCode (e.g. `glm-5.1-mxfp4-q8`).
- **`small_model`** should match **`model`** for heavy local models so OpenCode does not trigger a second on-demand load (titles, etc.) right after chat.
- **`reasoning`: `false`** for GLM in OpenCode avoids many “stuck waiting” issues with the compatible client.
- **Port:** `models.yaml` `server.port` and **`baseURL`** port must match (default **8000**).
- **One server process** per port; always **stop** the old one before starting again.

---

## Sanity check (any machine, server running)

```bash
curl -sS http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"glm-5.1-mxfp4-q8","messages":[{"role":"user","content":"Which is bigger, 1 or 10? One word."}],"stream":false,"max_tokens":32}'
```

If this returns JSON with a reply, the **bridge server** is healthy; any remaining issue is likely **OpenCode** or **model load time** on first on-demand request.
