# ATOMICAssetCore Community Edition

ATOMIC is a **knowledge graph + retrieval + inference system** that transforms raw datasets into structured, queryable intelligence.

It combines:
- Graph storage (Neo4j)
- Structured ingestion pipelines
- Retrieval-augmented inference
- Interactive chat interface

---

# 📚 Documentation Overview

This repository includes four core guides:

### 🚀 Quickstart Guide
End-to-end setup of the ATOMIC stack, including install, ingest, and inference.

➡️ `docs/quickstart.md`

---

### 🛠 Troubleshooting
Common errors, fixes, and debugging workflows.

➡️ `docs/troubleshooting.md`

---

### 📊 Corpus Cookbooks
Step-by-step guides for building high-quality datasets for ingestion.

➡️ `docs/corpus-cookbooks.md`

---

### 🔌 OpenClaw Integration
How to connect ATOMIC to OpenClaw for orchestration and external tooling.

➡️ `docs/openclaw-tutorial.md`

---

# ⚡ Quickstart (Condensed)

## 1. System Requirements

- macOS (Apple Silicon), Linux, or Windows (WSL)
- Docker + Docker Compose
- ~30GB free disk

Verify:

```bash
docker --version
docker compose --version
````

---

## 2. Download ATOMIC

Go to:

👉 [https://atomizer.ai/get-started](https://atomizer.ai/get-started)

Select your dataset and copy the install script.

---

## 3. Run Install Script

```bash
/bin/bash -c "$(curl -fsSL '<your-script-url>')"
```

This will:

* Authenticate with registry
* Pull containers
* Start:

  * Neo4j
  * ATOMIC API
  * Chat UI
* Begin dataset ingestion

---

## 4. Verify Services

### Neo4j

[http://localhost:7474](http://localhost:7474)
`neo4j / devpassword`

### API

[http://localhost:8880/docs](http://localhost:8880/docs)
(FastAPI UI should load)

### Chat UI

[http://localhost:8080](http://localhost:8080)

---

## 5. Confirm Everything is Running

```bash
docker ps
```

Expected:

* `atomic-neo4j`
* `atomic-community`
* `atomic-chat-web`

---

## ✅ Expected Result

You now have:

* A running knowledge graph
* A loaded dataset
* A queryable API
* A chat interface for interacting with your corpus

---

# 🧠 What to Do Next
* Checkout the full quickstart guide → `docs/quickstart.md `
* Learn ingestion workflows → `docs/corpus-cookbooks.md`
* Debug issues → `docs/troubleshooting.md`
* Integrate with external systems → `docs/openclaw-tutorial.md`

---

# 🛑 Stopping the Stack

```bash
docker compose down
```

Full reset:

```bash
docker compose down -v
```

---

# 💬 Support

If you run into issues:

* Check logs: `docker logs atomic-community`
* Use troubleshooting guide
* Reach out to the team via Discord or by starting a Github issue in this repo

---

# 🚀 Roadmap Notes

* Mac native binary (coming soon)
* GPU acceleration improvements
* Expanded ingestion pipelines
* Plugin ecosystem (OpenClaw + others)

```

---