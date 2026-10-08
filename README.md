# AtomicIQ documentation

AtomicIQ turns source material into a provenance-aware corpus for grounded search,
inference, and validation. **AtomicIQ** is the engine that packages the local
service and browser interface for local self-hosting, built from
[AtomicAppBuilder](https://github.com/NOLA-AI-Inc/AtomicAppBuilder)
(`ATOMIC_ENGINE=atomiciq`). On macOS it runs as a native menu-bar application
(**AtomicIQ.app**); on Windows (via WSL2) and Linux it runs as a Docker Compose
stack with a `llama.cpp` sidecar.

> Last verified: 2026-10-07 against AtomicIQ built from AtomicAppBuilder
> `main`. An edition may hide features that are not licensed or permitted for
> the signed-in user.

## Start with your goal

| Goal | Guide |
|---|---|
| Install the app and ask a first grounded question | [Quickstart](quickstart.md) |
| Understand Chat, Library, Validator, Account, and Settings | [Using AtomicIQ](app-guide.md) |
| Connect a trusted application to the local API | [Application integration](developer-workflow.md) |
| Look up authentication, CSRF, routes, and streaming events | [Local API reference](api-reference.md) |
| Prepare useful source material | [Corpus cookbooks](corpus-cookbooks.md) |
| Build owner-approved organizational knowledge | [Knowledge foundation](knowledge-foundation.md) |
| Diagnose startup, authentication, ingestion, or grounding | [Troubleshooting](troubleshooting.md) |

## System requirements

| Platform | Requirement |
|---|---|
| macOS | Apple Silicon (M-series), macOS 12+, at least 8 GB of unified memory. Inference runs against an external `llama.cpp` server attached over loopback (Metal-accelerated), not an in-process PyTorch/MLX model. |
| Windows (via WSL2) | Docker Desktop with WSL2. An NVIDIA GPU is recommended for the `llama.cpp` sidecar; AMD GPUs are not supported. |
| Linux | Docker + Docker Compose plugin. An NVIDIA GPU is recommended for the `llama.cpp` sidecar; AMD GPUs are not supported. |

CPU-only inference works for evaluation on Windows and Linux (drop the GPU
reservation and use a non-CUDA `llama.cpp` image) but is not recommended for
production workloads. All platforms need free disk space for the embedded
corpus store plus the downloaded GGUF model (a few GB). See
[Quickstart](quickstart.md) for per-platform install steps.

## How AtomicIQ works

```text
Source documents
      │
      ▼
Corpus ingestion ──→ provenance + knowledge tree + embeddings
      │                              │
      ├─→ Corpus catalog             ├─→ grounded Chat
      ├─→ Knowledge-tree navigation  └─→ claim validation
      └─→ reusable local knowledge, installable packs
```

AtomicIQ keeps these responsibilities separate:

| Layer | Responsibility |
|---|---|
| **Corpus** | Stores ingested documents, metadata, and provenance in an embedded LanceDB + RocksDB store |
| **Knowledge tree** | Retrieves compact, relevant context for inference (`corpus_toc`, `corpus_text`, `corpus_search`, …) |
| **Chat** | Produces an answer from retrieved evidence via an attached `llama.cpp` model and reports supporting context |
| **Validator** | Checks a draft or claim against corpus evidence |
| **Marketplace** | Installs certified knowledge packs onto the corpus shelf |
| **Account** | Manages the signed-in profile and per-user API keys |
| **Settings** | Validates and applies supported runtime or persistent configuration |

The model is a reasoning layer, not the source of truth. A defensible answer is
one whose evidence can be followed back to an ingested source.

## Local app and API

On macOS, launch **AtomicIQ** from Applications. The app starts the local
service and opens its interface in a browser. The preferred API port is `8880`,
but the launcher selects another free port when necessary. The effective URL is
recorded in:

```text
~/Library/Application Support/ATK/launcher_ports.env
```

On Windows (via WSL2) and Linux, AtomicIQ runs as a Docker Compose stack
(`atk-ee` plus a `llama-server` sidecar) and publishes the API on
`http://localhost:8880` by default — there is no `launcher_ports.env` to read.
Use `docker compose logs -f atk-ee` to watch startup.

The health endpoint is public:

```bash
curl -sS http://127.0.0.1:8880/v1/health
```

Use the effective URL from `launcher_ports.env` if that request cannot connect.
Authenticated state-changing requests also require AtomicIQ's CSRF handshake; see
the [Local API reference](api-reference.md) before writing a client.

## Trust rules

- Ingested text establishes what a source says, not whether the source is
  current or approved.
- Include ownership, scope, effective dates, and status in policy documents.
- Keep generated summaries distinguishable from authoritative documents.
- Never embed a personal API key in browser or mobile application code.
- Integrations should use the HTTP API, not the embedded store or files in
  the application-support directory.

## Support

Start with [Troubleshooting](troubleshooting.md). When reporting a problem,
include the effective local URL, failing route or app, HTTP status, and
relevant log excerpt. Remove API keys, cookies, credentials, and private
source text.

For documentation issues, open an issue in the
[documentation repository](https://github.com/NOLA-AI-Inc/ATOMIC-ToolKit-Community-Edition-Documentation).
