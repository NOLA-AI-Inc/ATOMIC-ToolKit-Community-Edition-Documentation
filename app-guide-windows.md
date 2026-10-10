# Using AtomicIQ on Windows

Platform-specific startup and local file details for Windows. For the shared
application walkthrough (Chat, Library, Validator, Marketplace, Curator,
Account, Settings), see [Using AtomicIQ](app-guide.md).

AtomicIQ on Windows has two installation paths today: the supported Docker
Compose stack (via WSL2), and an early native desktop app. Use Docker Compose
unless you are specifically testing the native build.

## Docker Compose (supported)

Install WSL2 and Docker Desktop, then run the same two-container stack as
Linux — see [Using AtomicIQ on Linux](app-guide-linux.md#install) for the
full `docker-compose.yml`, `.env`, and startup walkthrough; the steps and
files are identical on Windows once Docker Desktop's WSL2 integration is
enabled. See [Quickstart](quickstart.md#1-install-atomiciq) for the
WSL2/Docker Desktop setup steps.

## Native desktop app (preview)

AtomicIQ's desktop shell (`atomic-desktop`, built with Tauri) is
cross-platform and includes a Windows installer target (NSIS), the same
codebase that ships the macOS menu-bar app. As of this writing, the Windows
packaging is not yet at parity with macOS:

- There is no Windows equivalent of the macOS app bundle's `launch-atk.sh` /
  `stop-atk.sh` scripts, so the Windows build does not start the backend
  service or the `llama.cpp` sidecar for you. The window/tray shell attaches
  to a server that is already listening at `ATOMIC_DESKTOP_URL` (default
  `http://127.0.0.1:8880`) rather than launching one itself.
- There is no published Windows installer build yet — building one requires
  compiling `atomic-desktop` from source.
- Local AtomicIQ data on Windows falls back to a generic path,
  `%USERPROFILE%\ATK\`, rather than a Windows-conventional
  `%APPDATA%`/`%LOCALAPPDATA%` location — this is expected to change before
  the native build reaches parity with macOS.

Until this packaging work lands, treat the native Windows app as a
preview/developer build: start the backend yourself (for example, by running
the Docker Compose stack above and pointing `ATOMIC_DESKTOP_URL` at it, or by
running the Python service directly), then launch the Tauri shell to attach
to it. Check back here for updates as Windows packaging matures.

## Local files

The paths below apply when running the Docker Compose stack (the supported
path). See [Using AtomicIQ on Linux](app-guide-linux.md#local-files) — the
bind-mounted `data/` directory, container logs, and port behavior are
identical on Windows under WSL2.
