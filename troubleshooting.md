# Troubleshooting AtomicIQ

Start with the narrowest failing layer. A reachable health route does not prove
that authentication, corpus processing, or the attached `llama.cpp` inference
server is ready.

## Collect the basics

Before changing configuration, record:

- effective `ATOMIC_API_URL`;
- `inference_server.online` from `/v1/health`;
- the screen or API route that failed;
- HTTP status and response detail;
- whether the failure affects one user or every user; and
- a short, redacted excerpt from `atk.log`.

Never include API keys, access tokens, cookies, model credentials, or private
corpus text.

## The app does not open a browser

On macOS, AtomicIQ is a menu-bar application. Confirm its menu-bar item
is running, then inspect:

```bash
grep '^ATOMIC_API_URL=' "$HOME/Library/Application Support/ATK/launcher_ports.env"
tail -n 100 "$HOME/Library/Application Support/ATK/atk.log"
```

Open the recorded URL manually. The launcher prefers `8880` but automatically
selects another free port when necessary.

On Windows (WSL2) and Linux, AtomicIQ runs as a Docker Compose stack and
does not open a browser itself. Check container status and logs for both
services instead:

```bash
docker compose ps
docker compose logs -f atk-ee
docker compose logs -f llama-server
```

Open `http://localhost:8880` manually once `atk-ee` reports healthy; this
port is fixed by default (`ATK_PORT` in `.env`) and does not change between
runs.

On a first run, the `llama-server` sidecar will exit and restart repeatedly
with a "model not found" error until `atk-ee` finishes downloading the
configured GGUF into the shared `data/` volume. `restart: unless-stopped`
recovers automatically — use `docker compose logs -f atk-ee` to distinguish
that expected cold-start download from a genuinely failed process.

## Health endpoint cannot connect

On macOS:

```bash
ATOMICIQ_URL=$(sed -n 's/^ATOMIC_API_URL=//p' "$HOME/Library/Application Support/ATK/launcher_ports.env")
curl -sS --max-time 5 "$ATOMICIQ_URL/v1/health"
```

On Windows (WSL2) and Linux, the port is fixed by default:

```bash
curl -sS --max-time 5 http://localhost:8880/v1/health
```

If it fails:

1. On macOS, confirm the menu-bar app is running; on Windows/Linux, confirm
   the `atk-ee` container is running with `docker compose ps`.
2. On macOS, re-read `launcher_ports.env`; do not assume port `8880`. On
   Windows/Linux the port is `8880` unless you changed `.env`.
3. Inspect the end of `atk.log` (macOS) or
   `docker compose logs --tail 100 atk-ee` (Windows/Linux) for a
   startup or port error.
4. Restart from the app (macOS) or with `docker compose restart atk-ee`
   (Windows/Linux) after preserving any useful error details.
5. Confirm security software or a firewall is not blocking loopback
   connections.

## Health responds but `inference_server.online` is `false`

This means the HTTP service is up but the attached `llama.cpp` server is not
reachable — Chat and Tasks requests will fail or hang even though `/v1/health`
itself returns `200`.

- On a first run, wait for the GGUF download to finish (see "The app does not
  open a browser" above); this can take several minutes depending on
  connection speed.
- On macOS, confirm `community/llama_sidecar.py` has started a process
  listening on `127.0.0.1:8100`.
- On Windows/Linux, confirm the `llama-server` container is running and
  `docker compose logs -f llama-server` shows it serving, not restarting.
- Confirm `llm__llamacpp_gguf_repo` / `llm__llamacpp_gguf_filename` (or the
  Docker `LLAMACPP_GGUF_REPO` / `LLAMACPP_GGUF_FILENAME` variables) match
  between the app and the sidecar — a mismatch means the two sides disagree on
  which file should exist on disk.
- If `models__allow_hf_model_download=0`, confirm the GGUF and embedding
  model were copied into `~/Library/Application Support/ATK/models/`
  (or `./data/models/` for Docker) manually.

## The interface loads but login fails

- Confirm the local service is healthy.
- Check whether registration is restricted to organization-approved email
  addresses.
- Refresh the browser to obtain current session and CSRF state.
- Sign out and sign in again after an app restart or credential change.
- Inspect the response detail for `/api/v1/auth/*`; do not share its tokens.

## API returns `401 Unauthorized`

The request has no accepted session or API key, or the key was revoked.

- Create a fresh, dedicated key in **Account**.
- Send it as `Authorization: Bearer atk_...` or `X-API-Key: atk_...`.
- Confirm no shell quoting or whitespace altered the value.
- Verify the integration points to the same AtomicIQ instance where the
  key was created.

The complete key is displayed once. Losing it requires creating a replacement,
not recovering it from AtomicIQ storage.

## API returns `403 CSRF token missing`

The bearer credential may be valid. AtomicIQ also protects authenticated POST, PUT,
PATCH, and DELETE requests with a double-submit CSRF check.

Your client must:

1. retain the `fullauth_csrf` cookie issued by a safe GET;
2. send that cookie on the state-changing request; and
3. echo its value in `X-CSRF-Token`.

Follow the complete example in [Local API reference](api-reference.md). A stale
or rotated CSRF response should trigger one cookie refresh and one retry, not an
unbounded retry loop.

## API returns another `403`

The authenticated user may lack the required permission. Common boundaries are:

| Operation | Permission |
|---|---|
| Grounded Chat and Tasks | `inference:use` |
| Corpus reads | `corpus:view` |
| Upload, process, or clear corpus | `corpus:ingest` |
| View or change settings | settings permissions assigned by the server |

Do not work around permission failures by sharing a more powerful personal key.
Ask an administrator to grant the appropriate role or create an integration
identity consistent with organizational policy.

## Upload succeeds but the document is absent

An accepted upload can still be queued or fail during extraction.

1. Check Library → Explore → Catalog processing status.
2. Check `GET /v1/ingest/jobs` for the job's state.
3. Confirm the file type is PDF, DOCX, TXT, Markdown, a spreadsheet/CSV
   format, or JSON Lines.
4. Supply title, author, category, and publication date for ordinary documents.
5. Try a small, text-based source to isolate format from service health.
6. Inspect `atk.log` for extraction, embedding, or store errors.

Do not repeatedly upload the same large file until you understand whether the
first request is still processing.

## Grounded answers are empty or generic

- Confirm the source appears in the Library catalog.
- Confirm `inference_server.online` is `true`.
- Ask a question whose answer is unique to that source.
- Inspect the `citations` event rather than judging only the prose; check
  `answerable` and the returned `segments`.
- Add consistent metadata and categories.
- Split broad documents into coherent sections when retrieval is too diffuse.

If a source says nothing about the question, the correct result may be a
visible knowledge gap.

## Chat is slow or appears stuck

`llama.cpp` cold start, retrieval, and generation have different timing. A
client should stream the `crawl_status` phases, stream answer deltas, allow
cancellation, and use a longer idle timeout for the first request after
launch or a model download.

Check the log before restarting a working cold start. If every request stalls,
confirm `inference_server.online` and inspect Settings for the configured
`llm__llamacpp_base_url` and model files.

## Streaming text is malformed

`/v1/chat/stream/v2` returns Server-Sent Events, not one JSON document. Parse
each `data:` frame independently. Only `choices[0].delta.content` is assistant
text; `crawl_status`, `routing`, `carets`, and `citations` payloads have their
own shapes. End the stream on `[DONE]`.

See [Local API reference](api-reference.md) for current event types.

## A Settings change did not persist

- Confirm the field exists in `/v1/settings/schema`.
- Use the Settings screen or supported settings routes.
- Distinguish a runtime update from a persistent update.
- Restart AtomicIQ when the setting requires it.
- Reopen Settings and verify the effective value after restart.

Avoid manually adding unknown keys to `config.env`; unsupported names can make
startup validation fail.

## Find logs and configuration on macOS

```text
~/Library/Application Support/ATK/launcher_ports.env
~/Library/Application Support/ATK/config.env
~/Library/Application Support/ATK/models/
~/Library/Application Support/ATK/iq-stores/
~/Library/Application Support/ATK/atk.log
~/Library/Application Support/ATK/atk.*.log
```

The app may rotate logs. Capture the relevant time window soon after a failure.
Treat configuration and logs as sensitive because they can reveal local paths,
user details, provider names, or source identifiers.

## Find logs and configuration on Windows (WSL2) and Linux

Runtime state lives in the bind-mounted `data/` directory next to your
`docker-compose.yml`. There is no `launcher_ports.env` or `atk.log` file to
read directly; use Docker's own tooling instead:

```bash
docker compose logs --tail 100 atk-ee
docker compose logs --tail 100 llama-server
docker compose ps
```

Treat this log output as sensitive for the same reasons as the macOS files.

## Report a reproducible issue

Include:

```text
Platform (macOS / Windows WSL2 / Linux) and architecture:
effective AtomicIQ URL (no credentials):
inference_server.online (from /v1/health):
screen or HTTP method/path:
time of failure:
HTTP status and redacted detail:
expected result:
actual result:
smallest reproduction:
relevant redacted log lines:
```

State whether the same action succeeds in the shipped interface. That separates
an API/client problem from a service or corpus problem.
