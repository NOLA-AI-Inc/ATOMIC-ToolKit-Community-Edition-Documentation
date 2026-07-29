# ATōMIC ToolKit documentation

ATōMIC ToolKit (ATK) turns source material into a provenance-aware corpus for
grounded search, inference, and validation. ATOMIC Current packages the local
service and browser interface into a macOS menu-bar application.

> Last verified: 2026-07-22 against ATOMIC Current 1.422.13 with the 1.4.22
> backend. An edition may hide features that are not licensed or permitted for
> the signed-in user.

## Start with your goal

| Goal | Guide |
|---|---|
| Install the app and ask a first grounded question | [Quickstart](quickstart.md) |
| Understand Chat, Corpus, Validator, Account, and Settings | [Using ATOMIC Current](app-guide.md) |
| Connect a trusted application to the local API | [Application integration](developer-workflow.md) |
| Look up authentication, CSRF, routes, and streaming events | [Local API reference](api-reference.md) |
| Prepare useful source material | [Corpus cookbooks](corpus-cookbooks.md) |
| Build owner-approved organizational knowledge | [Knowledge foundation](knowledge-foundation.md) |
| Diagnose startup, authentication, ingestion, or grounding | [Troubleshooting](troubleshooting.md) |

## How the current product works

```text
Source documents
      │
      ▼
Corpus ingestion ──→ provenance + propositions + relationships
      │                              │
      ├─→ Corpus catalog             ├─→ grounded Chat
      ├─→ Graph exploration          └─→ claim validation
      └─→ reusable local knowledge
```

ATK keeps these responsibilities separate:

| Layer | Responsibility |
|---|---|
| **Corpus** | Stores ingested documents, metadata, and provenance |
| **Proposition/context layer** | Retrieves compact, relevant claims for inference |
| **Chat** | Produces an answer from retrieved evidence and reports supporting context |
| **Validator** | Checks a draft or claim against corpus evidence |
| **Graph and Calder** | Explore relationships or execute supported corpus expressions |
| **Account** | Manages the signed-in profile and per-user API keys |
| **Settings** | Validates and applies supported runtime or persistent configuration |

The model is a reasoning layer, not the source of truth. A defensible answer is
one whose evidence can be followed back to an ingested source.

## Local app and API

On macOS, launch **ATOMIC Current** from Applications. The app starts the local
service and opens its interface in a browser. The preferred API port is `8880`,
but the launcher selects another free port when necessary.

The effective URL is recorded in:

```text
~/Library/Application Support/ATK/launcher_ports.env
```

The health endpoint is public:

```bash
curl -sS http://127.0.0.1:8880/v1/health
```

Use the effective URL from `launcher_ports.env` if that request cannot connect.
Authenticated state-changing requests also require ATK's CSRF handshake; see
the [Local API reference](api-reference.md) before writing a client.

## Trust rules

- Ingested text establishes what a source says, not whether the source is
  current or approved.
- Include ownership, scope, effective dates, and status in policy documents.
- Keep generated summaries distinguishable from authoritative documents.
- Never embed a personal API key in browser or mobile application code.
- Integrations should use the HTTP API, not the embedded database or files in
  the application-support directory.

## Support

Start with [Troubleshooting](troubleshooting.md). When reporting a problem,
include the ATOMIC Current version, effective local URL, failing route or app,
HTTP status, and relevant log excerpt. Remove API keys, cookies, credentials,
and private source text.

For documentation issues, open an issue in the
[documentation repository](https://github.com/NOLA-AI-Inc/ATOMIC-ToolKit-Community-Edition-Documentation).
