# Using AtomicIQ on Linux

Platform-specific startup and local file details for Linux, which runs
AtomicIQ as a Docker Compose stack. For the shared application walkthrough
(Chat, Library, Validator, Marketplace, Curator, Account, Settings), see
[Using AtomicIQ](app-guide.md). For Windows, see
[Using AtomicIQ on Windows](app-guide-windows.md) — today it uses the same
Docker Compose stack described here, with a native app in preview.

## Start and reopen the interface

The compose stack runs two containers: `atk-ee` (the app) and `llama-server`
(the `llama.cpp` sidecar that serves chat/task inference). 

Run `docker compose up -d` to pull the published image and start the stack,
then open `http://localhost:8880` once `docker compose logs -f atk-ee` reports
the service healthy. On a cold start, `llama-server` will exit and restart
repeatedly with a "model not found" error until `atk-ee` finishes downloading
the configured GGUF into the shared `data/` volume — `restart: unless-stopped`
recovers automatically once the file exists. The published port is fixed by
`docker-compose.yml` (override with the `ATK_PORT` variable in `.env`), so
there is no launcher port file to read.

## Local files

The Docker Compose deployment keeps runtime state in the bind-mounted `data/`
directory next to your `docker-compose.yml`: downloaded models under
`data/models/`, persistent configuration in `data/config.env`, and the
embedded corpus store alongside them. There is no `launcher_ports.env`; the
API is always published on `http://localhost:8880` by default. Use
`docker compose logs -f atk-ee` for the application log and
`docker compose logs -f llama-server` for the inference sidecar, instead of
reading a file directly.
