# Building a knowledge foundation

Code, files, and model output can describe what exists. They do not tell AtomicIQ
what your organization has approved. A useful knowledge foundation supplies
that missing intent as scoped, owned, reviewable documents.

## What belongs in the corpus

High-value organizational sources include:

- architecture decisions and service boundaries;
- authentication, authorization, and trust-boundary rules;
- data ownership, retention, migration, and contract requirements;
- latency, capacity, and cost budgets;
- retry, failure, recovery, and observability expectations;
- required tests and release gates;
- compatibility, deprecation, and ownership policy; and
- product terminology and user-experience conventions.

Source code can help an author discover decisions that need documentation, but
it should not be promoted into policy merely because it is current.

## A safe document lifecycle

```text
Draft ──→ technical review ──→ owner approval ──→ active source ──→ AtomicIQ ingestion
  │                                                       │
  └──────────── never treated as active policy ───────────┘
```

1. Name an accountable owner.
2. Define scope and exclusions.
3. Write requirements as testable statements.
4. Record evidence, exceptions, and unresolved questions.
5. Obtain explicit approval and an effective date.
6. Ingest only the approved version through Corpus.
7. Supersede or remove it when policy changes.

Missing policy is not a violation. AtomicIQ should report the gap rather than invent
the organization's answer.

## Recommended Markdown format

AtomicIQ can ingest ordinary Markdown. Consistent structure makes retrieval and
human review stronger:

```markdown
# Service Authentication Standard

- Status: Active
- Owner: Platform Engineering
- Approved by: Name or review body
- Effective date: 2026-08-01
- Review by: 2027-02-01
- Applies to: Services in the production account
- Excludes: Local development tools
- Supersedes: AUTH-STD-001 revision 2

## Purpose
Why this standard exists.

## Definitions
Terms whose meaning must remain stable.

## Requirements

### AUTH-REQ-001 — Service identity
Every production service-to-service request must use ...

Evidence required: configuration receipt and integration test.
Exception owner: Security Engineering.

## Exceptions
How to request, approve, expire, and review an exception.

## Verification
How a reviewer can determine whether each requirement is met.

## Revision history
Material changes, dates, and approvers.
```

Stable requirement IDs let people cite a decision without depending on line
numbers that move during edits.

## Uploading an approved document

1. Open **Library** → **Add documents** in AtomicIQ.
2. Upload the active Markdown, PDF, DOCX, TXT, spreadsheet/CSV, or JSON Lines
   source.
3. Use a title that identifies the organization, subject, and version.
4. Set the accountable author or owner.
5. Choose a consistent category such as `security-standard` or
   `architecture-decision`.
6. Include the publication or effective date.
7. Wait for processing and confirm catalog presence.
8. Ask a question that cites a specific requirement ID and inspect provenance.

Keep drafts visibly marked inactive. If draft material must be searchable, use
a separate category and ensure applications do not present it as approved.

## Review checklist

- Is the owner named and still accountable?
- Is status explicit: draft, active, deprecated, or superseded?
- Are scope and exclusions concrete?
- Can each requirement be verified?
- Are exceptions owned and time-bounded?
- Does the document conflict with another active source?
- Is the revision or effective date clear?
- Would a retrieved section make sense without the rest of the file?

The [Corpus cookbooks](corpus-cookbooks.md) cover source preparation and
metadata in more detail.
