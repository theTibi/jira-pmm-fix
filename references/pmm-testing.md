# PMM testing commands (verified against `percona/pmm` tree)

Paths below are relative to a PMM clone root unless noted. Re-read `CONTRIBUTING.md` and component READMEs on your branch before copy-pasting.

## PMM Server devcontainer

From repo root (`Makefile`):

- `make env-up` — start devcontainer (`docker compose up -d --wait`).
- `make env-up-rebuild` — rebuild and start (after `env-update-image` path).
- `make env-down` — stop.

Inside the **pmm-server** container, `pmm-managed` iteration targets live in `Makefile.devcontainer` (e.g. `run-managed`, `run-managed-ci`). Confirm with:

```bash
grep -n run-managed Makefile.devcontainer
```

## api-tests (`api-tests/`)

Per [api-tests/README.md](https://github.com/percona/pmm/blob/main/api-tests/README.md):

1. Run PMM Server (`make env-up` from repo root).
2. Set `PMM_SERVER_URL` to `http://USERNAME:PASSWORD@HOST` (local dev often `http://admin:admin@127.0.0.1`).

Run tests:

```bash
cd api-tests
go test ./... -pmm.server-url "$PMM_SERVER_URL" -v
```

`api-tests/Makefile` also defines `make run-dev` (verbose `go test` with `-count=1 -p 1`) and package-scoped `build` targets for subdirs (`./alerting`, `./inventory`, etc.) — use those to narrow scope.

## UI (`ui/`)

Per [ui/README.md](https://github.com/percona/pmm/blob/main/ui/README.md):

- Prereqs: Node 22, Yarn.
- `make setup` — workspace setup.
- `make dev` — turborepo dev.
- `make build` — production build.
- From `package.json` scripts: `yarn test` runs `turbo run test` (Vitest in packages).

## Agent (`agent/`)

Per [agent/README.md](https://github.com/percona/pmm/blob/main/agent/README.md):

- `make env-up` for environment (from agent doc; aligns with server dev flow in monorepo).
- `make setup-dev` — configure agent against server.
- `make run` — run pmm-agent.
- `make test` — tests.
- `make check-all` — linters/checkers before PR.
