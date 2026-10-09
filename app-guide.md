# Using AtomicIQ

AtomicIQ combines the local service and its browser interface. On macOS, a
menu-bar process owns startup and shutdown; on Windows (via WSL2) and Linux, a
Docker Compose stack owns it instead. The browser provides the working
applications either way. Visible applications depend on edition and user
permissions — Community Edition excludes Curator, Marketplace, and Fleet.

## Start and reopen the interface

Startup, the interface URL, and local runtime files differ by platform:

- [Using AtomicIQ on macOS](app-guide-macos.md)
- [Using AtomicIQ on Windows](app-guide-windows.md)
- [Using AtomicIQ on Linux](app-guide-linux.md)

## The applications

### Chat

Chat sends a conversation through `POST /v1/chat/stream/v2` to the model
attached at the configured `llama.cpp` endpoint. In grounded mode, AtomicIQ
retrieves corpus context from the embedded knowledge tree before the model
answers. Supporting events may expose the crawl status, routing audit,
citations, and the corpus segments used during the turn.

For a meaningful grounding check:

1. Ask a question specific to an ingested document.
2. Inspect the supporting citations and source metadata.
3. Distinguish direct support from an inference that combines sources.
4. Treat absent evidence as a knowledge gap, not permission to invent a rule.

Some editions expose an ungrounded comparison mode. It is useful for testing,
but its answer should not be represented as corpus-backed.

### Library

Library is where you add, inspect, and manage source material, organized into
three tabs:

- **Shelves** — one card per subject area in this library, with a job queue
  for in-progress ingestion alongside. Create a **New shelf**, then use its
  **Add to shelf** action to upload files, a page, or a knowledge pack into
  it. Installed Marketplace packs also appear here as shelf cards.
- **Add documents** — upload files directly, or drop them into the watch
  folder when your edition has enabled it.
- **Explore** — split into **Graph explorer** (visualize the corpus and
  propositions graph) and **Catalog** (browse Documents, Shelves, Authors,
  Topics, Keywords, and Propositions via `GET /v1/corpus/catalog`, and search
  the library via `POST /v1/corpus/execute`).

The upload flow accepts PDF, DOCX, TXT, Markdown, spreadsheets/CSV, and
JSON Lines files, plus certified `.nola-pack` knowledge packs. Supply
descriptive metadata when the source does not already contain it.

Use the Catalog to confirm that processing completed. Clearing a corpus is
destructive and should be restricted to an authorized user who understands
which local knowledge will be removed.

If your installation has more than one store provisioned, use the store
selector in the header to switch the active corpus; see
[Local API reference](api-reference.md) for the underlying
`/v1/admin/store/select` route.

### Validator

Validator extracts claims from a draft and checks a selected claim against the
available corpus. A result reflects evidence AtomicIQ can currently retrieve; it does
not establish legal, organizational, or factual approval by itself.

Useful validation inputs are atomic and falsifiable. Split a sentence that
contains several independent claims before checking it.

### Marketplace

Marketplace (Enterprise Edition) is a storefront for certified knowledge packs:
browse the catalog, purchase or claim a pack, and install it onto the local
shelf. Installed packs appear in Library under **Shelves**, as shelf cards.
Purchases and entitlements are handled by a separate marketplace service that
the app proxies to; it does not run against your corpus data directly.

### Fleet

Fleet (Enterprise Edition, when the control plane is enabled) shows registered
installations and their status for an organization operating more than one
AtomicIQ deployment. It is off by default on a single desktop install.

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

## Configuration principles

- Use Settings when a supported field is available there.
- Back up `config.env` before a manual change.
- Restart the app after a persistent setting that requires it.
- Rediscover the effective ports and URL after every launch if your
  integration depends on automatic discovery — see the platform guide above
  for where each platform records this.
- Never place model, Atomizer, or AtomicIQ API credentials in documentation or a
  shared repository.

## What success looks like

A healthy local installation has four separate signals:

1. `/v1/health` responds, and its `inference_server.online` field is `true`.
2. The user can authenticate.
3. Corpus processing completes and the catalog contains the source.
4. A source-specific Chat or Validator request returns traceable citations.

Passing the first signal does not imply the other three.
