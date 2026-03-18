# ATOMICAssetCore Community Edition

## Quickstart Guide

## 1. System Requirements

### Required
- Linux (Ubuntu 22.04 recommended), macOS (arm64), or Windows (requires WSL)
- Docker
- Docker Compose
- 30+ GB free disk space  
  Additional space required varies based on the size of your corpus ingest.

### Verify
```bash
docker --version
docker compose --version
nvidia-container-cli info
````

## 2. Download the Image

Visit:

`https://atomizer.ai/get-started`

## 3. Choose Your Data Source

<!-- IMAGE: Insert JPEG from Quickstart PDF page 2 showing the "Choose Your Data Sources" UI -->

## 4. Run the Install Script

<!-- IMAGE: Insert JPEG from Quickstart PDF page 3 showing the install script screen -->

The install script will:

* Authenticate with the Atomizer container registry
* Download the ATOMIC Community containers
* Start the following services using Docker:

  * `atomic-neo4j`
  * `atomic-community`
  * `atomic-chat-web`

The script will also start ingestion of the datasets you selected within Atomizer.ai into your corpus on the running ATK stack.

## 5. Verify Services

### Neo4j

Open:

`http://localhost:7474`

Login:

* Username: `neo4j`
* Password: `devpassword`

### ATOMIC API

Open:

`http://localhost:8880/docs`

If you see the FastAPI docs, the stack is live.

### ATOMIC Chat Interface

Open:

`http://localhost:8080`

This launches the ATOMIC Chat UI, which connects to the running ATOMIC API and allows interactive querying of your corpus.

### Quick Service Check (CLI)

Verify containers are running:

```bash
docker ps
```

Expected containers:

* `atomic-neo4j`
* `atomic-community`
* `atomic-chat-web`

## Expected End State

After following this guide, you should have:

* ✅ Neo4j running
* ✅ ATOMIC API running
* ✅ Chat UI running
* ✅ Corpus cleared
* ✅ Example parquet ingested
* ✅ Inference returning grounded answers

---

# Advanced Guide

## 6. Clear Corpus

Use the API docs to locate the corpus management endpoint.

Endpoint pattern:

`POST /corpus/clear`

Run:

```bash
curl -X POST http://localhost:8880/corpus/clear
```

Confirm in Neo4j:

```cypher
MATCH (n) RETURN count(n);
```

This should return `0` or only system nodes.

## 7. Ingest Example Parquet

For more personalized example parquet generation using the Atomizer front-end corpus-ready export feature and dataset toolkit, see the Corpus Cookbooks.

### Step 7.1 — Create Example Parquet

On host:

```bash
python - <<'PY'
import pandas as pd
import pyarrow as pa
import pyarrow.parquet as pq

df = pd.DataFrame([
  {"doc_id":"1", "text":"ATOMIC builds AI knowledge graphs."},
  {"doc_id":"2", "text":"Neo4j stores relationships between entities."}
])

pq.write_table(pa.Table.from_pandas(df), "example.parquet")
print("Wrote example.parquet")
PY
```

### Step 7.2 — Copy Into Container

```bash
docker cp example.parquet atomic-community:/tmp/example.parquet
```

### Step 7.3 — Call Ingest Endpoint or Upload via Chat UI Web

Endpoint:

`POST /v1/ingest`

Example:

```bash
curl -X POST http://localhost:8880/v1/ingest \
  -H "Content-Type: application/json" \
  -d '{"path":"/tmp/example.parquet"}'
```

<!-- IMAGE: Insert JPEG from Quickstart PDF page 7 showing the ingest endpoint example and Chat UI screen -->

## 8. Confirm Ingest

In Neo4j Browser, run:

```cypher
MATCH (d:Document) RETURN d LIMIT 10;
```

You should see nodes representing ingested documents.

## 9. Run Inference Against Corpus via CLI

Find the inference endpoint in `/docs`.

Example:

```bash
curl -X POST http://localhost:8880/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "microsoft/Phi-3.5-mini-instruct",
    "messages": [
      {
        "role": "user",
        "content": "Explain what a REST API is in one sentence."
      }
    ],
    "top_k": 5
  }'
```

Expected behavior:

* Retrieval over ingested documents
* Grounded answer using the loaded model

## 9.5 Run Inference Using Chat UI

Open the chat interface:

`http://localhost:8080`

Enter a question such as:

> What is a REST API?

The Chat UI will send the request to the ATOMIC API and return a grounded answer based on your corpus.

<!-- IMAGE: Insert JPEG from Quickstart PDF page 9 showing the chat interface and sample prompt -->

## 9.6 Troubleshooting

If the API does not respond:

Check container status:

```bash
docker ps
```

View logs for the API container:

```bash
docker logs atomic-community
```

View logs for the chat UI:

```bash
docker logs atomic-chat-web
```

View Neo4j logs:

```bash
docker logs atomic-neo4j
```

<!-- IMAGE: Insert JPEG from Quickstart PDF page 10 showing troubleshooting examples -->

## 10. Stopping the Stack

```bash
docker compose down
```

Reset everything, including Neo4j data:

```bash
docker compose down -v
```
