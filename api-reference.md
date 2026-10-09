# Local API reference

AtomicIQ exposes a versioned HTTP API on the same local origin as its browser
interface. This reference covers the stable integration boundary used by the
app; an edition may omit routes that are not enabled.

> Verified against AtomicIQ built from AtomicAppBuilder `main` (2026-10-07).

## Discover the base URL

The preferred base URL is `http://127.0.0.1:8880`. On macOS, the launcher
chooses a different free port when required and writes the effective value to:

```text
~/Library/Application Support/ATK/launcher_ports.env
```

Example:

```bash
ATOMICIQ_URL=$(sed -n 's/^ATOMIC_API_URL=//p' "$HOME/Library/Application Support/ATK/launcher_ports.env")
curl -sS "$ATOMICIQ_URL/v1/health"
```

On Windows (via WSL2) and Linux, AtomicIQ runs as a Docker Compose stack that
publishes the API on `http://localhost:8880` by default (`ATK_PORT` in `.env`)
— there is no `launcher_ports.env` to read.

Treat `/v1/health` as reachability only. Most `/v1/*` routes require an
authenticated user. The response also carries `inference_server.online`,
which reflects whether the attached `llama.cpp` server is reachable. The
macOS default is `http://127.0.0.1:8100`; the Docker Compose setup uses
`http://llama-server:8080`. This is a separate signal from the HTTP
service being up.

## Authentication choices

### Browser session

The shipped browser interface authenticates through `/api/v1/auth/*`, refreshes
its session, and sends credentials on requests. Reuse that flow only for a
same-origin browser application designed to participate in AtomicIQ login.

### Per-user API key

Create an API key in **Account** for a trusted backend or local automation.
AtomicIQ accepts either header:

```http
Authorization: Bearer atk_...
```

or:

```http
X-API-Key: atk_...
```

The complete key is displayed once. Do not expose it in frontend JavaScript,
mobile binaries, logs, screenshots, or source control.

API key management routes are:

| Method | Route | Purpose |
|---|---|---|
| `GET` | `/v1/auth/api-keys` | List the current user's keys without secret values |
| `POST` | `/v1/auth/api-keys` | Create a key; the full value is returned once |
| `DELETE` | `/v1/auth/api-keys/{id}` | Revoke a key |

## CSRF for state-changing requests

Authenticated `POST`, `PUT`, `PATCH`, and `DELETE` requests require a
double-submit CSRF handshake in addition to authentication:

1. Send a safe GET request and retain the `fullauth_csrf` cookie.
2. Send that cookie on the mutation.
3. Copy its value into the `X-CSRF-Token` header.

A bearer key by itself is not enough. This is why a request can authenticate
and still return `403 {"detail":"CSRF token missing."}`.

### Complete curl example

```bash
ATOMICIQ_URL=$(sed -n 's/^ATOMIC_API_URL=//p' "$HOME/Library/Application Support/ATK/launcher_ports.env")
ATOMICIQ_API_KEY='replace-with-a-key-from-Account'
COOKIE_JAR=$(mktemp)

curl -sS -c "$COOKIE_JAR" "$ATOMICIQ_URL/v1/health" >/dev/null
CSRF=$(awk '$6 == "fullauth_csrf" { print $7 }' "$COOKIE_JAR")

curl -N -sS \
  -b "$COOKIE_JAR" \
  -H "Authorization: Bearer $ATOMICIQ_API_KEY" \
  -H "X-CSRF-Token: $CSRF" \
  -H 'Content-Type: application/json' \
  -d '{
    "message": "What does the corpus say about production access?"
  }' \
  "$ATOMICIQ_URL/v1/chat/stream/v2"
```

Delete the temporary cookie jar when finished. In production code, use a cookie
jar associated with the AtomicIQ base URL and refresh the CSRF cookie after a 403
that reports a stale or rotated token.

## Grounded chat

```http
POST /v1/chat/stream/v2
```

Representative request body:

```json
{
  "message": "What does the corpus require?",
  "messages": [],
  "k": 5,
  "temperature": 0.0,
  "session_id": "optional-stable-session-id",
  "output_task": "analytical"
}
```

`output_task` selects the response shape: `analytical` (default), `summary`,
or `continuation`. The response is Server-Sent Events (SSE). Clients should
handle:

| Event payload | Meaning |
|---|---|
| `crawl_status` | Retrieval phase progress (`vector` → `graph` → `gate`) |
| `bids` | Candidate-segment scoring (optional, audit) |
| `routing` | Routing/audit metadata for the turn |
| `choices[0].delta.content` | Incremental assistant text (OpenAI-compatible chunk format) |
| `carets` | Evidence-navigation audit data |
| `citations` | Final answer metadata: cited segments, `answerable`, supporting `segments` |
| `[DONE]` | End of stream |

Do not assume every data frame contains text. Ignore unknown event fields so a
new server can add metadata without breaking the client.

## Tasks

Use the tasks API for non-chat text operations that still run through the
retrieval-grounded dispatcher:

```http
POST /v1/tasks/execute
```

```json
{
  "task": "qa",
  "input_text": "...",
  "context": null,
  "target_domain": null,
  "options": {}
}
```

Response:

```json
{
  "task": "qa",
  "result_text": "...",
  "output_task": "analytical",
  "segments": [],
  "citations": [],
  "epistemic_analysis": {},
  "metadata": {}
}
```

Shortcut routes send the same response shape with the task name preset:

| Method | Route |
|---|---|
| `POST` | `/v1/tasks/summarize` |
| `POST` | `/v1/tasks/classify` |
| `POST` | `/v1/tasks/translate` |
| `POST` | `/v1/tasks/dialogue` |

## Corpus routes

| Method | Route | Purpose |
|---|---|---|
| `GET` | `/v1/corpus/stats` | Corpus counts and status |
| `GET` | `/v1/corpus/catalog` | Paginated catalog; supports facet filtering |
| `POST` | `/v1/ingest/file` | Multipart file upload |
| `GET` | `/v1/ingest/jobs` | List ingest job status |
| `POST` | `/v1/ingest/queue/kick` | Manually trigger ingest queue processing |
| `POST` | `/v1/corpus/execute` | Execute a knowledge-tree expression |
| `POST` | `/v1/clear-corpus` | Destructively clear corpus content |

The upload UI accepts PDF, DOCX, TXT, Markdown, spreadsheet/CSV, and JSON Lines
files, plus certified `.nola-pack` knowledge packs. Callers need the
permissions assigned by the server: corpus reads require `corpus:view`;
ingestion and clearing require `corpus:ingest`.

### `/v1/corpus/execute` expressions

```json
{ "expression": "corpus_toc('doc-id')" }
```

| Expression | Returns |
|---|---|
| `corpus_toc(rid_or_doc)` | Table of contents entries: `rid`, `label`, `snippet` |
| `corpus_text(rid)` | The paragraph/node text at `rid` |
| `corpus_parent(rid)` | `{ "parent_id": ..., "label": ..., "type": "document" }` |
| `corpus_adjacent(rid)` | `{ "prev": ..., "next": ... }` |
| `corpus_search(query)` | Formatted result lines matching `query` |

This is the AtomicIQ equivalent of ATOMIC Current's Calder navigator: same
route, a knowledge-tree-backed implementation instead of ArcadeDB.

## Store routes

| Method | Route | Purpose |
|---|---|---|
| `POST` | `/v1/admin/store/select` | Switch the active corpus store |

```json
{ "subject": "store-id" }
```

Switches the running process onto another provisioned store in the platform's
runtime data directory and persists the choice through `atomic-config`. It does
not re-index or materialize a new query-surface package — only an
already-provisioned store can be selected.

## Marketplace routes

| Method | Route | Purpose |
|---|---|---|
| `GET` | `/v1/marketplace/shelf` | Installed/available entitlements for this install |
| `POST` | `/v1/marketplace/packages/purchase` | Start a purchase for a catalog pack |
| `POST` | `/v1/marketplace/packages/install` | Install a purchased or free pack onto the shelf |
| `POST` | `/v1/packs/{subject}/import` | Import a pack file directly |

Installed packs appear in Library under **Shelves**, as shelf cards. Purchase
and entitlement routes proxy to a separate marketplace service; the
admin-only register/release routes require an operator token and are not
part of the end-user integration surface.

## Validation routes

| Method | Route | Request |
|---|---|---|
| `GET` | `/v1/demo/grounding-defaults` | None |
| `POST` | `/v1/demo/claim-extraction` | `{ "draft_text": "..." }` |
| `POST` | `/v1/demo/check-claim` | `{ "draft_text": "...", "claim": "..." }` |

These routes power Validator. Keep claims focused and retain the returned
evidence with any conclusion you share.

## Settings and logs

| Method | Route | Purpose |
|---|---|---|
| `GET` | `/v1/settings` | Current supported settings |
| `GET` | `/v1/settings/schema` | Field schema and validation information |
| `PATCH` | `/v1/settings/runtime` | Apply supported runtime updates |
| `PUT` | `/v1/settings/persistent` | Persist validated updates |
| `POST` | `/v1/settings/restart` | Restart after an authorized change |
| `GET` | `/v1/logs/tail` | Read a bounded recent log tail |
| `GET` | `/v1/logs/stream` | Stream logs when permitted |

Settings fields are grouped by domain: `knowledge` (corpus store),
`llm` / `models` / `serving` (the attached `llama.cpp` endpoint and model
choice), and `identity` / `ingest` / `query` / `packs` / `entitlements` /
`control_plane` (library behavior). Env vars follow `<domain>__<field>`, e.g.
`llm__llamacpp_base_url`, `ingest__parse_processes`. Prefer schema-driven
updates through Settings. Do not write directly to the embedded store.

## Compatibility and error handling

- Probe `/v1/health` at startup and check `inference_server.online` before
  assuming Chat or Tasks can complete.
- Treat `401` as missing, invalid, expired, or revoked authentication.
- Treat `403` as a permission or CSRF problem; inspect the response detail.
- Retry `429` and transient `5xx` responses with bounded exponential backoff.
- Give streaming requests an explicit cancellation path and a generous idle
  timeout for `llama.cpp` cold starts (first request after a model download).
- Preserve unknown JSON fields and SSE event types for forward compatibility.
