# OpenClaw integration

OpenClaw should integrate with AtomicIQ through the same supported local
HTTP API used by other trusted applications. Keep AtomicIQ credentials in a backend
tool or gateway; do not expose them in a browser-facing OpenClaw prompt or
client bundle.

## Recommended boundary

```text
OpenClaw agent
     │ allowlisted tool calls
     ▼
AtomicIQ adapter
     ├─ API key storage
     ├─ CSRF cookie lifecycle
     ├─ request limits and timeouts
     ├─ SSE parsing
     └─ citation normalization
     ▼
AtomicIQ /v1 API
```

The adapter should expose narrow operations such as:

- `atomiciq_health()`
- `atomiciq_search_corpus(query, limit)`
- `atomiciq_ask_grounded(message, session_id)`
- `atomiciq_check_claim(draft, claim)`

Do not expose corpus clearing, arbitrary settings changes, or unrestricted
expression execution unless the OpenClaw role explicitly requires and is
authorized for them.

## Setup

1. Start AtomicIQ and discover its effective base URL.
2. Create a dedicated key in **Account**.
3. Store the URL and key in the adapter's protected runtime configuration.
4. Implement the CSRF handshake described in the
   [Local API reference](api-reference.md).
5. Parse v2 Chat SSE events and return answer text separately from citation
   evidence.
6. Add timeouts, cancellation, and bounded retry behavior. Account for
   `llama.cpp` cold starts after launch or a model download.

## Tool-result contract

Return structured data to OpenClaw instead of a preformatted paragraph:

```json
{
  "grounded": true,
  "answer": "...",
  "sources": [
    { "title": "...", "identifier": "..." }
  ],
  "citations": [],
  "warnings": []
}
```

Set `grounded` to `false` when the `citations` event reports `answerable:
false` or no supporting segments. The agent should not convert an unsupported
answer into a grounded one through wording alone.

## Operational checks

- Verify health and `inference_server.online` before accepting work.
- Refresh CSRF once after a stale-token response.
- Treat revoked credentials as a hard stop.
- Surface model cold-start progress via the `crawl_status` events.
- Mark a disconnected SSE response incomplete.
- Log route, status, duration, and request ID when available, never the API key
  or private corpus content.

For the full client behavior, see [Application integration](developer-workflow.md).
