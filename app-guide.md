# Using ATOMIC Current

ATOMIC Current combines the local ATK service and its browser interface. The
menu-bar process owns startup and shutdown; the browser provides the working
applications. Visible applications depend on edition and user permissions.

## Start and reopen the interface

Launch **ATOMIC Current** from Applications and leave its menu-bar item running.
The launcher normally opens the interface automatically.

ATK prefers port `8880` and selects another free port when needed. The effective
URLs are recorded in:

```text
~/Library/Application Support/ATK/launcher_ports.env
```

Do not bookmark a guessed port on machines where other local services may use
it. Use the URL reported by the current launcher session.

## The applications

### Chat

Chat sends a conversation through the current inference pipeline. In grounded
mode, ATK retrieves corpus context before the model answers. Supporting events
may expose the Calder expression, result set, curiosity guidance, or tableaux
state used during the turn.

For a meaningful grounding check:

1. Ask a question specific to an ingested document.
2. Inspect the supporting context and source metadata.
3. Distinguish direct support from an inference that combines sources.
4. Treat absent evidence as a knowledge gap, not permission to invent a rule.

Some editions expose an ungrounded comparison mode. It is useful for testing,
but its answer should not be represented as corpus-backed.

### Corpus

Corpus is where you add, inspect, and manage source material. The current upload
flow accepts PDF, DOCX, TXT, Markdown, JSON, and Parquet files. Supply descriptive
metadata when the source does not already contain it.

Use the catalog and statistics to confirm that processing completed. Clearing a
corpus is destructive and should be restricted to an authorized user who
understands which local knowledge will be removed.

Some editions can exchange datasets with Atomizer. Configure the Atomizer key
in Account rather than embedding it in a document or client application.

### Validator

Validator extracts claims from a draft and checks a selected claim against the
available corpus. A result reflects evidence ATK can currently retrieve; it does
not establish legal, organizational, or factual approval by itself.

Useful validation inputs are atomic and falsifiable. Split a sentence that
contains several independent claims before checking it.

### Graph

Graph views expose corpus nodes, propositions, keywords, and nearby
relationships. They help users navigate the knowledge ATK has ingested. A graph
edge describes stored or derived structure; it does not prove causality.

### Calder

Calder executes supported corpus expressions and returns structured results.
Use it when you need a repeatable query rather than a prose answer. Keep the
expression and output together when sharing a result so another developer can
reproduce it.

### Curator

Curator supports workflows that refine or organize corpus material. Availability
and actions vary by edition. Review source provenance before accepting generated
or transformed content into an authoritative collection.

### Account

Account contains profile settings and per-user API keys. A newly created API key
is shown in full only once and uses the `atk_` prefix. Keys are stored hashed and
can be revoked.

Use a separate key per integration so access can be rotated without disrupting
other clients.

### Settings

Settings reads the server's current schema, validates supported values, and can
apply runtime or persistent updates. Prefer it over manually editing internal
state. Persistent changes may require an application restart.

## Local files on macOS

ATOMIC Current stores user-specific runtime state under:

```text
~/Library/Application Support/ATK/
```

Important files include:

| File | Purpose |
|---|---|
| `launcher_ports.env` | Effective API, WebSocket, and embedded-database ports |
| `config.env` | Persistent local configuration |
| `atk.log` | Current application log |
| `atk.*.log` | Rotated application logs |

These files are operational state, not an integration interface. External
applications should call the HTTP API instead of reading or modifying the
embedded database.

## Configuration principles

- Use Settings when a supported field is available there.
- Back up `config.env` before a manual change.
- Restart the app after a persistent setting that requires it.
- Read `launcher_ports.env` again after every launch if your integration depends
  on automatic port discovery.
- Never place model, Atomizer, or ATK API credentials in documentation or a
  shared repository.

## What success looks like

A healthy local installation has four separate signals:

1. `/v1/health` responds.
2. The user can authenticate.
3. Corpus processing completes and the catalog contains the source.
4. A source-specific Chat or Validator request returns traceable evidence.

Passing the first signal does not imply the other three.
