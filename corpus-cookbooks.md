# Corpus Cookbooks

Practical recipes for turning raw source material into a corpus AtomicIQ can
ground Chat and Tasks against. There is no separate dataset-authoring tool —
you ingest files directly through the **Library** app's **Add documents** tab
or `POST /v1/ingest/file`, the same way for every source type below.

Use [Knowledge foundation](knowledge-foundation.md) when the source is an
organizational standard or policy. Ownership, scope, status, and effective
dates matter as much as the text itself.

## Supported file types

- Markdown / plain text (`.md`, `.txt`)
- PDF (`.pdf`)
- Word documents (`.docx`)
- Spreadsheets (`.xlsx`, `.xls`, `.ods`, `.csv`, `.tsv`)
- JSON Lines (`.jsonl`)
- Certified knowledge packs (`.nola-pack`)

AtomicIQ parses each file into the embedded corpus store (LanceDB + RocksDB +
OverGraph) — there is no intermediate export/approval step and no row-level
template authoring. Prepare the file so it reads cleanly, then ingest it.

## General workflow

1. Convert or export your source material into one of the supported file
   types above.
2. Split very large or multi-topic sources into smaller files — one file per
   article, chapter, or document works better for retrieval than one giant
   file.
3. Add a short title/date/author header at the top of each file (or use
   spreadsheet columns for the same fields) so AtomicIQ can surface
   attribution alongside answers.
4. Open **Library → Add documents** in the app (or call `POST /v1/ingest/file`) and
   upload the file(s). Use `POST /v1/clear-corpus` first if you are replacing
   an existing corpus rather than adding to it.
5. Validate with a question whose answer is unique to the ingested source,
   and confirm the response reports grounding/citations rather than only
   plausible prose — see [Local API reference](api-reference.md) for the
   `citations` field shape.

---

## Cookbook A: Substack articles → writing assistant

### Goal

Build a corpus that can answer questions about your past articles and
continue writing in your voice.

### Prepare

Export your posts (Substack's export gives HTML/Markdown per post) and save
each article as its own `.md` file, named after the post slug. Put the title,
subtitle, and publish date at the top of each file:

```markdown
# The title of the post

_Subtitle — published 2026-01-14_

Body text of the article starts here...
```

### Ingest

Upload the folder of `.md` files via **Library → Add documents**. Reject or fix
articles with broken formatting or truncated bodies before uploading — clean
input produces a cleaner corpus than a bulk dump.

### Result

A corpus that supports style imitation, topic recall, and article
referencing in Chat.

---

## Cookbook B: Academic slides → study bot

### Goal

Create a test-prep assistant grounded in lecture material.

### Prepare

Export lecture decks as PDF (most slide tools support this directly). Where
slides are sparse ("Agenda", title-only slides), either delete them from the
export or merge them with the following content slide so each page carries
real information.

### Ingest

Upload the PDFs via **Library → Add documents**, one file per lecture or course
module. Add course name and subject as part of the filename or a short header
line so retrieval can be filtered/attributed by course later.

### Result

A retrieval-aware study assistant that can answer concept questions and
summarize lecture content with citations back to the source deck.

---

## Cookbook C: Large reference corpus

### Goal

Ingest a large body of reference material (an internal wiki export, a
documentation set, or a public corpus you already have locally as text/PDF).

### Prepare

Keep the natural document boundaries from the source — one file per page or
chapter — rather than concatenating everything into a single file. Very large
single files are slower to retrieve against precisely and make citations
coarser.

### Ingest

Upload in batches via **Library → Add documents** (or script it against
`POST /v1/ingest/file`) and check `GET /v1/ingest/jobs` for progress on each
batch. See [Integration guide](developer-workflow.md) for the scripted
ingestion flow.

### Result

A broad knowledge base suitable for grounded Chat and Tasks across the whole
document set.

---

## Best practices across all cookbooks

1. **Clean input beats clever prompting.** Fix broken formatting, truncated
   text, and empty files before ingesting rather than relying on the model to
   compensate.
2. **One topic per file.** Splitting by article/chapter/slide-deck gives
   sharper retrieval and citations than one large merged file.
3. **Carry metadata in the file.** A short title/date/author header (or
   spreadsheet columns) is the only attribution AtomicIQ has — add it before
   ingesting, not after.
4. **Separate corpora by use case.** Don't mix writing-style material,
   academic reference material, and general web content in the same corpus
   store if you need distinct behavior — use
   `POST /v1/admin/store/select` to manage multiple stores.
