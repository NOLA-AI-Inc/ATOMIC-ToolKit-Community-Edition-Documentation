# Using AtomicIQ on macOS

Platform-specific startup and local file details for macOS. For the shared
application walkthrough (Chat, Corpus, Validator, Marketplace, Curator,
Account, Settings), see [Using AtomicIQ](app-guide.md).

## Start and reopen the interface

Launch **AtomicIQ** (Community Edition builds show as **AtomicIQ Community
Edition**) from Applications and leave its menu-bar item running. A Tauri
window (`atomic-desktop`) stays hidden until the local service reports
healthy, then opens the interface automatically. The menu bar's Start/Stop
actions run `launch-atk.sh` / `stop-atk.sh` from the app bundle.

AtomicIQ prefers port `8880` and selects another free port when needed. The
effective URLs are recorded in:

```text
~/Library/Application Support/ATK/launcher_ports.env
```

Do not bookmark a guessed port on machines where other local services may use
it. Use the URL reported by the current launcher session.

Unlike ATOMIC Current, AtomicIQ does not run a model in-process. The app
attaches to an external `llama.cpp` `llama-server` process at
`127.0.0.1:8100` (Metal-accelerated on Apple Silicon) for Chat and Tasks; the
corpus store (LanceDB + RocksDB + OverGraph) runs embedded, so there is no
separate database process to start.

## Local files

AtomicIQ stores user-specific runtime state under the same directory as
ATOMIC Current — the two engines share one config file:

```text
~/Library/Application Support/ATK/
```

Important files include:

| File | Purpose |
|---|---|
| `launcher_ports.env` | Effective API and WebSocket ports |
| `config.env` | Persistent local configuration (`knowledge__`, `llm__`, `models__`, `serving__`, `identity__`, `ingest__`, … domains) |
| `models/` | Downloaded embedding model and GGUF weights (`models__allow_hf_model_download=1` by default) |
| `iq-stores/<id>/` | Embedded corpus store(s); switch the active one with Settings or `/v1/admin/store/select` |
| `atk.log` | Application log |
| `atk.*.log` | Rotated application logs |

These files are operational state, not an integration interface. External
applications should call the HTTP API instead of reading or modifying the
embedded store.
