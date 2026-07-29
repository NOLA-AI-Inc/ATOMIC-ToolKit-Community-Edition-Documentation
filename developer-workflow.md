# Application integration

Use ATOMIC Current's HTTP API when another application needs grounded answers,
corpus search, validation, or ingestion. Do not couple an integration to the
embedded database, application bundle, or implementation-specific files.

## Choose the right integration shape

| Client | Recommended design |
|---|---|
| ATK's own browser interface | Same-origin login session managed by the app |
| Trusted local automation | Per-user API key, cookie jar, and CSRF handshake |
| Server-side product | Backend adapter that protects the API key and normalizes ATK responses |
| Browser or mobile product | Its backend calls ATK; never ship the ATK key to the client |

The safest reusable boundary is a small backend adapter. It can discover the
local endpoint, protect credentials, retain cookies, parse SSE, enforce request
timeouts, and expose only the ATK operations your product needs.

## Integration flow

```text
Your UI
  │
  ▼
Your trusted backend adapter
  ├─ discovers ATK URL
  ├─ holds per-integration API key
  ├─ maintains CSRF cookie
  ├─ sends versioned /v1 requests
  └─ parses SSE and evidence events
  │
  ▼
ATOMIC Current API ──→ corpus retrieval ──→ grounded response
```

## 1. Establish readiness

At process startup:

1. Discover the effective URL.
2. Call `GET /v1/health` with a short timeout.
3. Record the reported version when present.
4. Obtain the CSRF cookie before the first mutation.
5. Test one authenticated read that matches the client's permissions.

On macOS, a local companion can read the effective URL from:

```text
~/Library/Application Support/ATK/launcher_ports.env
```

For a remote or managed deployment, make the base URL explicit configuration.
Do not assume `8880` will always be free.

## 2. Create a least-privilege key

Create a distinct API key in **Account** for each integration. Name it for the
consumer and environment, store it outside source control, and revoke it when
the integration is retired.

The server applies permissions to the authenticated user. A client that only
asks grounded questions does not need corpus-clearing or settings-management
capabilities.

## 3. Implement authentication and CSRF together

All authenticated state-changing requests need both:

- `Authorization: Bearer atk_...` or `X-API-Key: atk_...`; and
- the `fullauth_csrf` cookie echoed as `X-CSRF-Token`.

Implement this once in the adapter's request layer. Do not teach each feature
to recreate the handshake. See the [Local API reference](api-reference.md) for
a complete request.

## 4. Stream grounded chat correctly

`POST /v1/chat/stream/v2` returns SSE. A robust client:

- appends only `choices[0].delta.content` to visible assistant text;
- retains `calder_result`, `curiosity_guide`, and `tableaux_data` as evidence or
  inspection metadata;
- displays `queue_status` without treating it as answer text;
- terminates on `[DONE]`, `cancelled`, or a fatal `error`;
- ignores unknown event fields;
- supports user cancellation; and
- uses a long enough idle timeout for local model startup.

Keep the final answer and its evidence metadata associated in application
state. Stripping the grounding events makes later review less trustworthy.

## 5. Integrate ingestion deliberately

Use the Corpus app for human-driven uploads. Use `/v1/ingest/file` or the
directory-processing routes only when an automated source has clear ownership,
metadata, and update semantics.

Before automating ingestion, decide:

- how a source is identified and deduplicated;
- who owns its accuracy;
- whether it is authoritative, advisory, generated, or historical;
- how updates and removals are represented;
- which users may ingest it; and
- how a failed or partially processed upload is surfaced.

Do not turn every application record into corpus text. Choose source material
that will remain understandable when retrieved outside its original screen.

## 6. Preserve provenance in your UI

Show users:

- whether a response was grounded or bare;
- which source titles or identifiers supported it;
- whether evidence was missing or conflicting;
- the time and ATK version associated with the request; and
- a clear boundary between quoted source material and model synthesis.

If the product needs a decisive business rule, ingest the approved rule with
scope and ownership rather than encoding it only in a prompt.

## 7. Design for failure

| Failure | Client behavior |
|---|---|
| ATK is not running | Explain how to start Current; do not spin indefinitely |
| `401` | Stop and request key/session repair |
| CSRF `403` | Refresh the cookie once, then surface the failure |
| Permission `403` | Report the required operation; do not retry |
| `429` | Retry with bounded exponential backoff |
| Model cold start | Show progress and allow cancellation |
| SSE disconnect | Mark the answer incomplete; never present it as final |
| No grounding evidence | Label the response unsupported or ask for a better source |

## 8. Verify before release

Use a small integration test corpus containing facts that are absent from the
base model. Verify:

1. health and authentication;
2. CSRF renewal;
3. a grounded answer and its evidence events;
4. a deliberately unsupported question;
5. cancellation and timeout handling;
6. a revoked API key; and
7. compatibility with an automatically selected port.

The API contract is documented in [Local API reference](api-reference.md).
