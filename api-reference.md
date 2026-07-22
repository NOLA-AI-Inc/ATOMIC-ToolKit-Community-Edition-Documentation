# Local API reference

ATOMIC Current exposes a versioned HTTP API on the same local origin as its
browser interface. This reference covers the stable integration boundary used
by the current app; an edition may omit routes that are not enabled.

> Verified against ATOMIC Current 1.422.13 with backend 1.4.22.

## Discover the base URL

The preferred base URL is `http://127.0.0.1:8880`. The macOS launcher chooses a
different free port when required and writes the effective value to:

```text
~/Library/Application Support/ATK/launcher_ports.env
```

Example:

```bash
ATK_URL=$(sed -n 's/^ATOMIC_API_URL=//p' "$HOME/Library/Application Support/ATK/launcher_ports.env")
curl -sS "$ATK_URL/v1/health"
```

Treat `/v1/health` as reachability only. Most `/v1/*` routes require an
authenticated user.

## Authentication choices

### Browser session

The shipped browser interface authenticates through `/api/v1/auth/*`, refreshes
its session, and sends credentials on requests. Reuse that flow only for a
same-origin browser application designed to participate in ATK login.

### Per-user API key

Create an API key in **Account** for a trusted backend or local automation. ATK
accepts either header:

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
ATK_URL=$(sed -n 's/^ATOMIC_API_URL=//p' "$HOME/Library/Application Support/ATK/launcher_ports.env")
ATK_API_KEY='replace-with-a-key-from-Account'
COOKIE_JAR=$(mktemp)

curl -sS -c "$COOKIE_JAR" "$ATK_URL/v1/health" >/dev/null
CSRF=$(awk '$6 == "fullauth_csrf" { print $7 }' "$COOKIE_JAR")

curl -N -sS \
  -b "$COOKIE_JAR" \
  -H "Authorization: Bearer $ATK_API_KEY" \
  -H "X-CSRF-Token: $CSRF" \
  -H 'Content-Type: application/json' \
  -d '{
    "messages": [{"role": "user", "content": "What does the corpus say about production access?"}],
    "update_corpus_carat": true
  }' \
  "$ATK_URL/v1/chat/stream/v2"
```

Delete the temporary cookie jar when finished. In production code, use a cookie
jar associated with the ATK base URL and refresh the CSRF cookie after a 403
that reports a stale or rotated token.

## Grounded chat

Use the v2 streaming route for new integrations:

```http
POST /v1/chat/stream/v2
```

Representative request body:

```json
{
  "model": "optional-model-name",
  "messages": [
    { "role": "user", "content": "What does the corpus require?" }
  ],
  "update_corpus_carat": true,
  "session_id": "optional-stable-session-id",
  "system_prompt": "optional-integration-specific-instruction"
}
```

The response is Server-Sent Events (SSE). Clients should handle:

| Event payload | Meaning |
|---|---|
| `choices[0].delta.content` | Incremental assistant text |
| `calder_result` | Grounding expression and structured result |
| `curiosity_guide` | Additional evidence-navigation guidance |
| `tableaux_data` | Updated tableaux state |
| `queue_status` | Current queue position |
| `cancelled` | Server-side cancellation |
| `error` | Structured request failure |
| `[DONE]` | End of stream |

Do not assume every data frame contains text. Ignore unknown event fields so a
new server can add metadata without breaking the client.

`POST /v1/chat/stream/v2/bare` provides an ungrounded comparison path. Do not
label its output corpus-backed. `POST /v1/chat/completions` remains a legacy
non-streaming compatibility route; prefer v2 for new work.

## Corpus routes

| Method | Route | Purpose |
|---|---|---|
| `GET` | `/v1/corpus/stats` | Corpus counts and status |
| `GET` | `/v1/corpus/catalog` | Paginated catalog; supports facet filtering |
| `POST` | `/v1/ingest/file` | Multipart file upload |
| `POST` | `/v1/process-files` | Process configured source directories |
| `GET` | `/v1/directories/status` | Directory processing status |
| `POST` | `/v1/corpus/execute` | Execute a supported corpus expression |
| `POST` | `/v1/clear-corpus` | Destructively clear corpus content |

The upload UI currently accepts PDF, DOCX, TXT, Markdown, JSON, and Parquet.
Callers need the permissions assigned by the server: corpus reads require
`corpus:view`; ingestion and clearing require `corpus:ingest`.

## Graph and retrieval routes

| Method | Route | Purpose |
|---|---|---|
| `GET` | `/v1/graph/propositions` | Search propositions by term |
| `GET` | `/v1/graph/corpus/search` | Search corpus nodes |
| `GET` | `/v1/graph/corpus/root` | Retrieve a bounded root projection |
| `GET` | `/v1/graph/corpus/node/{rid}` | Expand a corpus node |

Use bounded limits and depth. Graph results describe ATK's stored knowledge,
not the live state of an unrelated source system.

## Validation routes

| Method | Route | Request |
|---|---|---|
| `GET` | `/v1/demo/grounding-defaults` | None |
| `POST` | `/v1/demo/claim-extraction` | `{ "draft_text": "..." }` |
| `POST` | `/v1/demo/check-claim` | `{ "draft_text": "...", "claim": "..." }` |

These routes power current validation experiences. Keep claims focused and
retain the returned evidence with any conclusion you share.

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

Prefer schema-driven updates through Settings. Do not write directly to the
embedded database.

## Compatibility and error handling

- Probe `/v1/health` at startup and record the returned version when available.
- Treat `401` as missing, invalid, expired, or revoked authentication.
- Treat `403` as a permission or CSRF problem; inspect the response detail.
- Retry `429` and transient `5xx` responses with bounded exponential backoff.
- Give streaming requests an explicit cancellation path and a generous idle
  timeout for local model cold starts.
- Do not depend on `/v1/knowledge/*` in v2 mode; use corpus, graph, validation,
  and v2 Chat routes.
- Preserve unknown JSON fields and SSE event types for forward compatibility.
