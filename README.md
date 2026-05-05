# cc-edge-pack-template

Template repository for new Cribl Edge / Stream packs in the
[dryvist](https://github.com/dryvist) org. Provides a TypeScript + Vitest test
harness, Biome lint/format, reusable GitHub Actions workflows, and a Makefile
that wraps the common day-to-day commands.

## What's in the box

- **Test harness** — Vitest tests that validate route + pipeline behavior against
  a live Cribl container (auto-discovered fixtures, no test code edits to add cases)
- **CI** — reusable GitHub Actions workflows for validate + test + release
- **Lint/format** — Biome with pre-commit hook (lefthook)
- **Release** — release-please-driven version bumps + `.crbl` artifact upload

## Installation

Create a new pack repo from this template:

```sh
gh repo create dryvist/cc-edge-<source>-io \
  --template dryvist/cc-edge-pack-template \
  --public --clone

cd cc-edge-<source>-io
make install            # installs Node deps + git hooks
```

`make install` requires `node` (20+) and `pnpm` (10+). Docker is only needed
when you run `make test`. See [`docs/development.md`](docs/development.md) for
installation paths.

## After scaffolding

Replace template placeholders before opening your first PR:

- `package.json` — `name`, `displayName`, `description`, `tags`, `author`
- `default/pack.yml` — `displayName`, `description`
- `.github/workflows/test.yml` — flip `pack_type:` from `edge` to `stream` if
  this is a Cribl Stream pack (default is `edge`)

Then replace the demo passthrough pipeline (`default/pipelines/passthrough/`)
and its fixture (`tests/fixtures/passthrough/`) with your pack's real
pipelines and fixtures.

## Usage

```sh
make docker-up          # start cribl/cribl test container
make test               # vitest run (auto-discovers fixtures)
make typecheck          # tsc --noEmit
make lint               # biome check
make format             # biome format --write
make build              # build .crbl artifact
make docker-down        # stop test container
```

Add a fixture under `tests/fixtures/<pipeline>/<case>.json` and `make test`
picks it up automatically — no code changes required.

## Releases

Don't tag versions manually. release-please opens a PR with the computed
version bump on every push to `main`; merge that PR to publish a release. See
[`docs/release-process.md`](docs/release-process.md).

## Documentation

| Doc | What it covers |
|---|---|
| [`docs/development.md`](docs/development.md) | Local dev setup, Make targets, optional Nix shell |
| [`docs/test-harness.md`](docs/test-harness.md) | What gets tested, fixture conventions |
| [`docs/file-boundary.md`](docs/file-boundary.md) | Generic vs pack-specific files (sync rules) |
| [`docs/release-process.md`](docs/release-process.md) | release-please flow, version bump rules |
| [`docs/validator-rules.md`](docs/validator-rules.md) | vct-cribl-pack-validator rules + how they're enforced |
| [`.github/README.md`](.github/README.md) | Reusable workflows reference |

## Contributing

Conventional commits required (`feat:`, `fix:`, `chore:`, `docs:`). Branches
must start with `feat/`, `fix/`, `chore/`, etc. (org ruleset). PRs against
`main` are squash-merged.

Changes here propagate to every downstream pack — tread carefully. See
[`AGENTS.md`](AGENTS.md) and [`docs/file-boundary.md`](docs/file-boundary.md).

## License

Apache-2.0 — see [`LICENSE`](LICENSE).
