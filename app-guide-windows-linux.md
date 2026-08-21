# Using ATOMIC Current on Windows and Linux

Platform-specific startup and local file details for Windows (via WSL2) and
Linux. For the shared application walkthrough (Chat, Corpus, Validator, Graph,
Calder, Curator, Account, Settings), see
[Using ATOMIC Current](app-guide.md).

## Start and reopen the interface

Run `docker compose up -d` and open `http://localhost:8880` once
`docker compose logs -f atomic-current` reports the service healthy. This port
is fixed by `docker-compose.yml`, so there is no launcher port file to read.

## Local files

The Docker Compose deployment keeps runtime state in the bind-mounted `data/`
directory next to your `docker-compose.yml`, including the ArcadeDB database
under `data/arcadedb_data`. There is no `launcher_ports.env`; the API is always
published on `http://localhost:8880`. Use
`docker compose logs -f atomic-current` for the application log instead of
reading a file directly.
