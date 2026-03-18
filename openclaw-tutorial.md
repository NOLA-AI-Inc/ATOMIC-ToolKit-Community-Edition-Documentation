# OpenClaw Tutorial

## Part 1 — Stand up ATK (Docker)

### 1. Pull / Start the ATK Image

```bash
docker compose pull
docker compose up -d
````

### 2. Verify the API is Up

```bash
curl -s http://localhost:8000/v1/health
```

Open the docs UI:

* `http://localhost:8000/docs`
* OpenAPI JSON: `http://localhost:8000/openapi.json`

### 3. Quick Sanity Test: Call ATK Completions

```bash
curl -s http://localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "prompt":"Say hello in one sentence.",
    "model":"nola-ai/token_expanded_phi_3.5",
    "update_corpus_carat": true,
    "enhance_embeddings": true
  }'
```

The request fields above are compatible with the OpenAPI schema.

---

## Part 2 — Stand up OpenClaw (Node)

### 1. Install + Run the Setup Wizard

OpenClaw’s getting started flow is:

```bash
npm install -g openclaw@latest
openclaw onboard
```

### 2. Start the Gateway

```bash
openclaw gateway start
```

They also document:

```bash
openclaw gateway run
```

for foreground debugging.

### 3. Confirm Your Keys Live in `~/.openclaw/.env`

OpenClaw recommends using environment variables stored in `~/.openclaw/.env`, including:

* `OPENAI_API_KEY`
* and/or `ANTHROPIC_API_KEY`

They also recommend locking permissions:

```bash
chmod 600 ~/.openclaw/.env
```

---

## Part 3 — Integrate ATK into the OpenClaw “Stack” (Demo That Works Today)

### Important Constraint

Based on the OpenClaw docs:

* the Anthropic provider supports `apiKey`
* Minimax shows `baseUrl` + `api` mode

But the provided OpenClaw docs do **not** document an `openai.baseUrl` override, so this tutorial does **not** assume you can point OpenClaw’s OpenAI provider directly at `http://localhost:8000`.

### What We’ll Do Instead

We’ll run OpenClaw normally and add ATK as a sidecar inference service you can call from:

* a terminal script today
* a real OpenClaw plugin later

This still demonstrates:

> OpenClaw + existing LLM stack + ATK OpenAPI inference component

---

## Part 3A — Add an ATK Connector Script

Create `atk_chat.sh`:

```bash
cat > atk_chat.sh <<'SH'
#!/usr/bin/env bash
set -euo pipefail

ATK_URL="${ATK_URL:-http://localhost:8000}"
MODEL="${ATK_MODEL:-nola-ai/token_expanded_phi_3.5}"

PROMPT="${1:-}"
if [[ -z "$PROMPT" ]]; then
  echo "Usage: $0 'your prompt here'" >&2
  exit 1
fi

curl -s "$ATK_URL/v1/chat/completions" \
  -H "Content-Type: application/json" \
  -d "{
    \"prompt\": $(python - <<'PY'
import json,sys
print(json.dumps(sys.argv[1]))
PY
"$PROMPT"),
    \"model\": \"$MODEL\",
    \"update_corpus_carat\": true,
    \"enhance_embeddings\": true
  }"
SH

chmod +x atk_chat.sh
```

### Why This Works

* It calls ATK’s documented endpoint: `POST /v1/chat/completions`
* It uses the documented request schema fields

Try it:

```bash
./atk_chat.sh "Give me 3 bullet points about how to study for a biology exam."
```

---

## Part 3B — Use OpenClaw + ATK Together

Pattern:

* OpenClaw is your orchestrator, channels, memory, and tools layer
* ATK is a dedicated inference/corpus service

### Workflow

1. Message OpenClaw in Telegram, Discord, or another supported channel.
2. For corpus-aware answers, or for your ATK-specific model, call ATK via `atk_chat.sh` today.
3. Later, replace the script with a plugin wrapper.

### Manual Demo Prompt Format

In OpenClaw, send:

> I’m going to call ATK for a corpus-aware answer. My question: X. When I paste the ATK output, incorporate it and cite it.

Then run:

```bash
./atk_chat.sh "X"
```

Paste the result back into OpenClaw.

That is enough for an end-to-end tutorial that works without hidden assumptions.

---

## Next Step (Optional): Turn the Connector into a Real OpenClaw Plugin

Your docs show how plugins are configured and installed:

* npm source
* version pinning
* enable/disable

What the provided excerpts do **not** include is the plugin API contract, meaning:

* what functions a plugin must implement
* what files it must expose

If you add that section later, this connector can be turned into a proper local plugin, for example:

```bash
openclaw plugins add ./openclaw-atk
```

Potential plugin features:

* an `/atk` command
* a tool that OpenClaw can invoke automatically

---

## Quickstart Checklist

### 1. Start ATK

```bash
cd ATOMICAssetCore/community-edition
docker compose pull
docker compose up -d
curl -s http://localhost:8000/v1/health
```

### 2. Start OpenClaw

```bash
npm install -g openclaw@latest
openclaw onboard
openclaw gateway start
```

### 3. Call ATK Inference

```bash
./atk_chat.sh "Hello from the integrated stack."
```


Source: OpenClaw PDF and pasted OpenClaw tutorial text. :contentReference[oaicite:7]{index=7}