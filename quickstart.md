# ATOMIC Current quickstart

This guide starts the local app, creates a useful corpus, and verifies a
grounded answer.

## 1. Install ATOMIC Current

ATOMIC Current requires macOS 12 or newer. Download the edition supplied by
your organization or visit [atomizer.ai/get-started](https://atomizer.ai/get-started),
then move **ATOMIC Current.app** to Applications.

Allow at least 30 GB of free disk space, plus space for source documents and
downloaded models.

## 2. Launch the app

Open **ATOMIC Current** from Applications. It runs as a menu-bar application,
starts the local service, and opens the interface in your default browser.

The first startup can take several minutes while ATK initializes its embedded
database or obtains a configured model. Keep the menu-bar app running while you
use the interface.

The preferred URL is:

```text
http://127.0.0.1:8880/
```

If that port is busy, the launcher chooses another one. On macOS, read the
effective URL with:

```bash
grep '^ATOMIC_API_URL=' "$HOME/Library/Application Support/ATK/launcher_ports.env"
```

## 3. Create or sign in to your account

Complete the registration or sign-in screen shown by the app. Registration may
be limited to email addresses approved by your organization.

The browser interface uses a local authenticated session. API keys are created
separately in **Account** and should be used only by trusted integrations.

## 4. Add source material

Open **Corpus** and upload a supported file. Current builds accept:

- PDF (`.pdf`)
- Word (`.docx`)
- text (`.txt`)
- Markdown (`.md`)
- JSON (`.json`)
- Parquet (`.parquet`)

For documents without embedded corpus metadata, supply a useful title, author,
category, and publication date. Prefer a small, coherent first document whose
contents you know well.

Wait for ingestion to finish, then confirm the document appears in the Corpus
catalog or statistics. An accepted upload does not necessarily mean every
document has finished processing.

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
ATK_URL=http://127.0.0.1:8880
curl -sS "$ATK_URL/v1/health"
```

Use the value from `launcher_ports.env` when ATK selected a different port.
Health proves that HTTP is reachable. It does not prove that a user is signed
in, a corpus is ready, or a grounded model request can complete.

## 8. Create an API key only when needed

In **Account**, create a per-user API key for a trusted local or server-side
integration. The complete `atk_...` value is displayed once. Store it in a
secret manager or protected environment variable and revoke it when no longer
needed.

Do not paste a key into source code or ship it in a browser bundle. ATK's
authenticated POST, PUT, PATCH, and DELETE routes also require a CSRF cookie and
matching `X-CSRF-Token` header. Follow [Application integration](developer-workflow.md)
and the [Local API reference](api-reference.md).

## Next steps

- Learn each screen in [Using ATOMIC Current](app-guide.md).
- Prepare stronger documents with [Corpus cookbooks](corpus-cookbooks.md).
- Add organizational context with [Knowledge foundation](knowledge-foundation.md).
- Diagnose a problem with [Troubleshooting](troubleshooting.md).
