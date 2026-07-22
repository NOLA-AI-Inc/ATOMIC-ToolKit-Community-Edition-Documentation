# Troubleshooting ATOMIC Current

Start with the narrowest failing layer. A reachable health route does not prove
that authentication, corpus processing, or grounded inference is ready.

## Collect the basics

Before changing configuration, record:

- ATOMIC Current version;
- effective `ATOMIC_API_URL`;
- the screen or API route that failed;
- HTTP status and response detail;
- whether the failure affects one user or every user; and
- a short, redacted excerpt from `atk.log`.

Never include API keys, access tokens, cookies, model credentials, or private
corpus text.

## The app does not open a browser

ATOMIC Current is a menu-bar application. Confirm its menu-bar item is running,
then inspect:

```bash
grep '^ATOMIC_API_URL=' "$HOME/Library/Application Support/ATK/launcher_ports.env"
tail -n 100 "$HOME/Library/Application Support/ATK/atk.log"
```

Open the recorded URL manually. The launcher prefers `8880` but automatically
selects another free port when necessary.

On a first run, model or database initialization may take several minutes. The
launcher can wait substantially longer than an ordinary web request, so use the
log to distinguish cold startup from a failed process.

## Health endpoint cannot connect

```bash
ATK_URL=$(sed -n 's/^ATOMIC_API_URL=//p' "$HOME/Library/Application Support/ATK/launcher_ports.env")
curl -sS --max-time 5 "$ATK_URL/v1/health"
```

If it fails:

1. Confirm the menu-bar app is running.
2. Re-read `launcher_ports.env`; do not assume port `8880`.
3. Inspect the end of `atk.log` for a startup or port error.
4. Restart from the app after preserving any useful error details.
5. Confirm security software is not blocking loopback connections.

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
- Verify the integration points to the same ATOMIC Current instance where the
  key was created.

The complete key is displayed once. Losing it requires creating a replacement,
not recovering it from ATK storage.

## API returns `403 CSRF token missing`

The bearer credential may be valid. ATK also protects authenticated POST, PUT,
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
| Grounded Chat | `inference:use` |
| Corpus and graph reads | `corpus:view` |
| Upload, process, or clear corpus | `corpus:ingest` |
| View or change settings | settings permissions assigned by the server |

Do not work around permission failures by sharing a more powerful personal key.
Ask an administrator to grant the appropriate role or create an integration
identity consistent with organizational policy.

## Upload succeeds but the document is absent

An accepted upload can still be queued or fail during extraction.

1. Check Corpus processing status and statistics.
2. Confirm the file type is PDF, DOCX, TXT, Markdown, JSON, or Parquet.
3. Supply title, author, category, and publication date for ordinary documents.
4. Try a small, text-based source to isolate format from service health.
5. Inspect `atk.log` for extraction, embedding, or database errors.

Do not repeatedly upload the same large file until you understand whether the
first request is still processing.

## Grounded answers are empty or generic

- Confirm the source appears in the Corpus catalog.
- Ask a question whose answer is unique to that source.
- Inspect grounding/evidence events rather than judging only the prose.
- Check that the source has enough local context to make retrieved sections
  understandable.
- Add consistent metadata and categories.
- Split broad documents into coherent sections when retrieval is too diffuse.

If a source says nothing about the question, the correct result may be a
visible knowledge gap.

## Chat is slow or appears stuck

Local model startup, queueing, retrieval, and generation have different timing.
A client should display `queue_status`, stream answer deltas, allow
cancellation, and use a longer idle timeout for the first request after launch.

Check the log before restarting a working cold start. If every request stalls,
inspect Settings for the selected inference backend and model configuration.

## Streaming text is malformed

`/v1/chat/stream/v2` returns Server-Sent Events, not one JSON document. Parse
each `data:` frame independently. Only `choices[0].delta.content` is assistant
text; grounding, queue, tableaux, cancellation, and error payloads have their
own shapes. End the stream on `[DONE]`.

See [Local API reference](api-reference.md) for current event types.

## `/v1/knowledge/*` returns `503`

The knowledge routes belong to the older v1 knowledge service and may be
unavailable when Current runs the v2 inference pipeline. Use the current
Corpus, Graph, Validator, and `/v1/chat/stream/v2` routes instead.

## A Settings change did not persist

- Confirm the field exists in `/v1/settings/schema`.
- Use the Settings screen or supported settings routes.
- Distinguish a runtime update from a persistent update.
- Restart ATOMIC Current when the setting requires it.
- Reopen Settings and verify the effective value after restart.

Avoid manually adding unknown keys to `config.env`; unsupported names can make
startup validation fail.

## Find logs and configuration on macOS

```text
~/Library/Application Support/ATK/launcher_ports.env
~/Library/Application Support/ATK/config.env
~/Library/Application Support/ATK/atk.log
~/Library/Application Support/ATK/atk.*.log
```

The app may rotate logs. Capture the relevant time window soon after a failure.
Treat configuration and logs as sensitive because they can reveal local paths,
user details, provider names, or source identifiers.

## Report a reproducible issue

Include:

```text
ATOMIC Current version:
macOS version and architecture:
effective ATK URL (no credentials):
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
