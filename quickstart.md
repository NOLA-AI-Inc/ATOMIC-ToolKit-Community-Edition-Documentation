# AtomicIQ quickstart

This guide starts the local app, creates a useful corpus, and verifies a
grounded answer.

## 1. Install AtomicIQ

Visit [atomizer.ai/get-started](https://atomizer.ai/get-started) and pick your
platform.

### macOS

Requires macOS 12 or newer on Apple Silicon (M-series). Download the `.pkg`
supplied by your organization and run the installer. Confirm **AtomicIQ.app**
(Community Edition builds show as **AtomicIQ Community Edition.app**) is in
Applications. Allow a few GB of free disk space for the downloaded `llama.cpp`
model plus space for source documents.

### Windows (via WSL2)

1. Open PowerShell **as Administrator** and install WSL2 with Ubuntu:

   ```powershell
   wsl --install
   ```

   Restart your computer when prompted.
2. Launch **Ubuntu** from the Start Menu and create a UNIX username/password.
3. Install [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/),
   then in Settings enable:
   - General → "Use the WSL 2 based engine"
   - Resources → WSL Integration → enable your Ubuntu distro
4. Run every command below from the **Ubuntu** terminal, not PowerShell or
   Command Prompt.

An NVIDIA GPU is recommended for the `llama.cpp` sidecar that serves chat and
task inference; AMD GPUs are not supported. CPU-only inference works for
evaluation (drop the GPU reservation and switch to a non-CUDA `llama.cpp`
image) but is not recommended for production workloads.

### Linux

Requires Docker and the Docker Compose plugin. An NVIDIA GPU is recommended
for the `llama.cpp` sidecar; AMD GPUs are not supported. Install the NVIDIA
Container Toolkit (`nvidia-smi` should work inside containers) if you want GPU
acceleration.

### Windows and Linux: run the Docker Compose stack

See [Using AtomicIQ on Linux](app-guide-linux.md#install) for the full
`docker-compose.yml`, `.env`, and first-boot steps — the same containers and
files apply on Windows under WSL2. Self-hosting requires accepting the
license agreement shown at
[atomizer.ai/get-started](https://atomizer.ai/get-started).

## 2. Launch the app

On macOS, open **AtomicIQ** from Applications. It runs as a menu-bar
application, starts the local service, and opens the interface in your default
browser.

On Windows and Linux, once `atk-ee` reports healthy in the compose logs, open
the interface directly in your browser.

The first startup can take several minutes while AtomicIQ initializes its embedded
corpus store and the `llama.cpp` sidecar downloads its model. Keep the
menu-bar app (macOS) or the Docker stack (Windows/Linux) running while you use
the interface.

The preferred URL is:

```text
http://127.0.0.1:8880/
```

On macOS, if that port is busy the launcher chooses another one. Read the
effective URL with:

```bash
grep '^ATOMIC_API_URL=' "$HOME/Library/Application Support/ATK/launcher_ports.env"
```

On Windows and Linux, the Docker Compose stack publishes the API on
`http://localhost:8880` by default (override with `ATK_PORT` in `.env`) — there
is no `launcher_ports.env` to read.

## 3. Create or sign in to your account

Complete the registration or sign-in screen shown by the app. Registration may
be limited to email addresses approved by your organization.

The browser interface uses a local authenticated session. API keys are created
separately in **Account** and should be used only by trusted integrations.

## 4. Add source material

Open **Library** → **Add documents** and upload a supported file. AtomicIQ
builds accept:

- PDF (`.pdf`)
- Word (`.docx`)
- text (`.txt`)
- Markdown (`.md`)
- spreadsheets/CSV (`.xlsx`, `.xls`, `.ods`, `.csv`, `.tsv`)
- JSON Lines (`.jsonl`)
- certified knowledge packs (`.nola-pack`, installed from **Marketplace**,
  appearing under **Library** → **Shelves**)

For documents without embedded corpus metadata, supply a useful title, author,
category, and publication date. Prefer a small, coherent first document whose
contents you know well.

Wait for ingestion to finish, then confirm the document appears under
**Library** → **Explore** → **Catalog**. An accepted upload does not
necessarily mean every document has finished processing.

## 5. Ask a grounded question

Open **Chat** and ask a question whose answer exists only in the document you
just uploaded. For example:

```text
According to the onboarding policy, who approves production access and what
evidence is required?
```

Inspect the cited context or provenance. A generic question is a weak test
because a model may answer it without using your corpus.

## 6. Try claim validation

Open **Validator**, paste a short draft, and extract or check one factual claim.
Validation compares the claim with available corpus evidence; it is not a
substitute for an accountable reviewer.

## 7. Verify the service from a terminal

The health route does not require authentication:

```bash
ATOMICIQ_URL=http://127.0.0.1:8880
curl -sS "$ATOMICIQ_URL/v1/health"
```

Use the value from `launcher_ports.env` when AtomicIQ selected a different port.
Health proves that HTTP is reachable and reports whether the `llama.cpp`
inference server is online. It does not prove that a user is signed in, a
corpus is ready, or a grounded model request can complete.

## 8. Create an API key only when needed

In **Account**, create a per-user API key for a trusted local or server-side
integration. The complete `atk_...` value is displayed once. Store it in a
secret manager or protected environment variable and revoke it when no longer
needed.

Do not paste a key into source code or ship it in a browser bundle. AtomicIQ's
authenticated POST, PUT, PATCH, and DELETE routes also require a CSRF cookie and
matching `X-CSRF-Token` header. Follow [Application integration](developer-workflow.md)
and the [Local API reference](api-reference.md).

## Next steps

- Learn each screen in [Using AtomicIQ](app-guide.md).
- Prepare stronger documents with [Corpus cookbooks](corpus-cookbooks.md).
- Add organizational context with [Knowledge foundation](knowledge-foundation.md).
- Diagnose a problem with [Troubleshooting](troubleshooting.md).
