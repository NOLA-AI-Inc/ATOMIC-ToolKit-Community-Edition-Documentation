# Repository source material

ATK can make repository documentation searchable, but it does not inspect a
live checkout merely because the repository exists on the same machine. The
corpus contains only material that has been deliberately uploaded or processed
through a supported ingestion path.

## Useful repository sources

- README and onboarding guides;
- architecture decision records;
- API and data-contract documentation;
- operational runbooks;
- approved engineering standards;
- release notes and migration plans; and
- generated code summaries whose status and revision are explicit.

Generated summaries are implementation evidence, not approved organizational
policy. Label them with the repository, commit or release, generation date, and
tool that produced them.

## Recommended metadata

| Field | Example |
|---|---|
| Title | `Payments API overview at release 2026.07` |
| Author | `Payments team` |
| Category | `repository-documentation` |
| Revision | `commit 4f29c1a` |
| Status | `generated`, `reviewed`, or `active` |
| Scope | `services/payments` |
| Publication date | `2026-07-22` |

Place this metadata in the document and in the Corpus upload fields where
available.

## Keep answers revision-aware

When asking ATK about code documentation, include the repository and revision
in the question. If the source revision is older than the code under review,
treat the answer as historical context and inspect the current repository with
normal development tools.

ATK's corpus graph describes relationships in ingested knowledge. It is not a
replacement for a compiler, language server, test suite, or live source-code
index.
