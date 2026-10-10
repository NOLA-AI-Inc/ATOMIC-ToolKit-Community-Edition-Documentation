# Using AtomicIQ on Linux

Docker Compose install, startup, and local file details for Linux, which runs
AtomicIQ as a Docker Compose stack. For the shared application walkthrough
(Chat, Library, Validator, Marketplace, Curator, Account, Settings), see
[Using AtomicIQ](app-guide.md). For Windows, see
[Using AtomicIQ on Windows](app-guide-windows.md) — today it uses the same
Docker Compose stack described here, with a native app in preview.

## Install

Requires Docker and the Docker Compose plugin. An NVIDIA GPU is recommended
for the `llama.cpp` sidecar; AMD GPUs are not supported. Install the NVIDIA
Container Toolkit (`nvidia-smi` should work inside containers) if you want GPU
acceleration. CPU-only inference works for evaluation (drop the GPU
reservation and switch to a non-CUDA `llama.cpp` image) but is not
recommended for production workloads. Self-hosting requires accepting the
license agreement shown at
[atomizer.ai/get-started](https://atomizer.ai/get-started).

AtomicIQ ships as two containers: the app itself (`atk-ee`) and a `llama.cpp`
`llama-server` sidecar that serves the chat/task model. There is no ArcadeDB
service to run — the corpus store (LanceDB + RocksDB + OverGraph) is embedded
in the app container.

1. Create a project directory and save the following as `docker-compose.yml`.
   This pulls the published image — no local build or source checkout
   required. (Community Edition does not include Marketplace/Fleet/Curator;
   swap the image for `ghcr.io/nola-ai-inc/atomic-community-edition` if that's
   your entitlement.)

   ```yaml
   services:
     atk-ee:
       image: ghcr.io/nola-ai-inc/atomic-enterprise-edition:${TAG:-latest}
       environment:
         ATOMIC_ENGINE: atomiciq
         models__hf_token: ${HF_TOKEN:-}
         LLAMA_SERVER_HOST: ${LLAMA_SERVER_HOST:-llama-server}
         llm__llamacpp_gguf_repo: ${LLAMACPP_GGUF_REPO:-openbmb/MiniCPM5-2B-GGUF}
         llm__llamacpp_gguf_filename: ${LLAMACPP_GGUF_FILENAME:-MiniCPM5-2B-Q8_0.gguf}
       volumes:
         - ./data:/app/data
       ports:
         - "${ATK_BIND_HOST:-127.0.0.1}:${ATK_PORT:-8880}:8880"
       deploy:
         resources:
           reservations:
             devices:
               - driver: nvidia
                 count: 1
                 capabilities: [gpu]
       depends_on:
         - llama-server
       restart: unless-stopped

     llama-server:
       image: ${LLAMACPP_IMAGE:-ghcr.io/ggml-org/llama.cpp:server-cuda}
       hostname: ${LLAMA_SERVER_HOST:-llama-server}
       networks:
         default:
           aliases:
             - ${LLAMA_SERVER_HOST:-llama-server}
       command:
         - --model
         - /app/data/models/${LLAMACPP_GGUF_FILENAME:-MiniCPM5-2B-Q8_0.gguf}
         - --host
         - "0.0.0.0"
         - --port
         - "8080"
         - --ctx-size
         - "${LLAMACPP_CTX_SIZE:-8192}"
         - --n-gpu-layers
         - "${LLAMACPP_N_GPU_LAYERS:-999}"
       volumes:
         - ./data:/app/data:ro
       # llama-server has no authentication of its own. atk-ee reaches it over
       # the internal Compose network (llama-server:8080), so no host port is
       # published by default. Uncomment to expose it for local debugging —
       # keep it loopback-only, never bind it to a non-localhost interface:
       # ports:
       #   - "127.0.0.1:${LLAMA_SERVER_PORT:-18100}:8080"
       deploy:
         resources:
           reservations:
             devices:
               - driver: nvidia
                 count: 1
                 capabilities: [gpu]
       restart: unless-stopped
   ```

2. In the same directory, save the following as `.env` and adjust if needed.
   The defaults download `openbmb/MiniCPM5-2B-GGUF` (`MiniCPM5-2B-Q8_0.gguf`)
   with an 8192-token context window. Set `HF_TOKEN` only if your chosen
   repo/filename is gated on Hugging Face.

   `ATK_BIND_HOST` defaults to `127.0.0.1`, so the UI/API port is only reachable
   from the Docker host itself — Docker otherwise publishes a port on every
   network interface, which would expose AtomicIQ to your whole LAN. If you are
   deploying to a server and want to reach it from other machines, set
   `ATK_BIND_HOST=0.0.0.0` (or a specific interface IP) here, ideally behind a
   reverse proxy or firewall rule that restricts who can connect.

   ```bash
   TAG=latest
   ATK_PORT=8880
   ATK_BIND_HOST=127.0.0.1
   LLAMA_SERVER_HOST=llama-server
   LLAMA_SERVER_PORT=18100
   LLAMACPP_IMAGE=ghcr.io/ggml-org/llama.cpp:server-cuda
   LLAMACPP_GGUF_REPO=openbmb/MiniCPM5-2B-GGUF
   LLAMACPP_GGUF_FILENAME=MiniCPM5-2B-Q8_0.gguf
   LLAMACPP_CTX_SIZE=8192
   LLAMACPP_N_GPU_LAYERS=999
   # HF_TOKEN=
   ```
3. Start the stack:

   ```bash
   docker compose up -d
   ```
4. Watch startup (first boot downloads the GGUF into the shared `./data`
   volume; the `llama-server` container will exit and restart repeatedly with
   a "model not found" error until that download finishes — this is expected):

   ```bash
   docker compose logs -f atk-ee
   ```
5. Once `atk-ee` reports healthy, open `http://localhost:8880` (or your
   `ATK_BIND_HOST`/`ATK_PORT` override) in your browser.

## Start and reopen the interface

The compose stack runs two containers: `atk-ee` (the app) and `llama-server`
(the `llama.cpp` sidecar that serves chat/task inference).

After the initial install above, run `docker compose up -d` from the project
directory any time to (re)start the stack, then open `http://localhost:8880`
once `docker compose logs -f atk-ee` reports the service healthy. The
published port is fixed by `docker-compose.yml` (override with the
`ATK_BIND_HOST`/`ATK_PORT` variables in `.env`), so there is no launcher port
file to read.

## Local files

The Docker Compose deployment keeps runtime state in the bind-mounted `data/`
directory next to your `docker-compose.yml`: downloaded models under
`data/models/`, persistent configuration in `data/config.env`, and the
embedded corpus store alongside them. There is no `launcher_ports.env`; the
API is always published on `http://localhost:8880` by default. Use
`docker compose logs -f atk-ee` for the application log and
`docker compose logs -f llama-server` for the inference sidecar, instead of
reading a file directly.
