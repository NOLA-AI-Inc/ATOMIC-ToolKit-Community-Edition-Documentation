# Using ATOMIC Current on macOS

Platform-specific startup and local file details for macOS. For the shared
application walkthrough (Chat, Corpus, Validator, Graph, Calder, Curator,
Account, Settings), see [Using ATOMIC Current](app-guide.md).

## Start and reopen the interface

Launch **ATOMIC Current** from Applications and leave its menu-bar item
running. The launcher normally opens the interface automatically.

ATK prefers port `8880` and selects another free port when needed. The
effective URLs are recorded in:

```text
~/Library/Application Support/ATK/launcher_ports.env
```

Do not bookmark a guessed port on machines where other local services may use
it. Use the URL reported by the current launcher session.

## Local files

ATOMIC Current stores user-specific runtime state under:

```text
~/Library/Application Support/ATK/
```

Important files include:

| File | Purpose |
|---|---|
| `launcher_ports.env` | Effective API, WebSocket, and embedded-database ports |
| `config.env` | Persistent local configuration |
| `atk.log` | Current application log |
| `atk.*.log` | Rotated application logs |

These files are operational state, not an integration interface. External
applications should call the HTTP API instead of reading or modifying the
embedded database.
