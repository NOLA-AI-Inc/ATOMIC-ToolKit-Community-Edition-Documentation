# OpenClaw integration

OpenClaw should integrate with ATOMIC Current through the same supported local
HTTP API used by other trusted applications. Keep ATK credentials in a backend
tool or gateway; do not expose them in a browser-facing OpenClaw prompt or
client bundle.

## Recommended boundary

```text
OpenClaw agent
     │ allowlisted tool calls
     ▼
ATK adapter
     ├─ API key storage
     ├─ CSRF cookie lifecycle
     ├─ request limits and timeouts
     ├─ SSE parsing
     └─ evidence normalization
     ▼
ATOMIC Current /v1 API
```

The adapter should expose narrow operations such as:

- `atk_health()`
- `atk_search_corpus(query, limit)`
- `atk_ask_grounded(messages, session_id)`
- `atk_check_claim(draft, claim)`

Do not expose corpus clearing, arbitrary settings changes, or unrestricted
expression execution unless the OpenClaw role explicitly requires and is
authorized for them.

## Setup

1. Start ATOMIC Current and discover its effective base URL.
2. Create a dedicated key in **Account**.
3. Store the URL and key in the adapter's protected runtime configuration.
4. Implement the CSRF handshake described in the
   [Local API reference](api-reference.md).
5. Parse v2 Chat SSE events and return answer text separately from grounding
   evidence.
6. Add timeouts, cancellation, and bounded retry behavior.

## Tool-result contract

Return structured data to OpenClaw instead of a preformatted paragraph:

```json
{
  "grounded": true,
  "answer": "...",
  "sources": [
    { "title": "...", "identifier": "..." }
  ],
  "evidence": [],
  "warnings": [],
  "atk_version": "..."
}
```

Set `grounded` to `false` when the bare endpoint was used or supporting corpus
evidence was absent. The agent should not convert an unsupported answer into a
grounded one through wording alone.

## Operational checks

- Verify health before accepting work.
- Refresh CSRF once after a stale-token response.
- Treat revoked credentials as a hard stop.
- Surface queue and model-startup progress.
- Mark a disconnected SSE response incomplete.
- Log route, status, duration, and request ID when available, never the API key
  or private corpus content.

For the full client behavior, see [Application integration](developer-workflow.md).
