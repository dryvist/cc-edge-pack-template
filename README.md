# cc-edge-pack-template

Template repository for new Cribl Edge / Stream packs in the dryvist org. Provides the full DRY scaffolding (TypeScript test harness, validation, release packaging, Makefile, Docker setup, Nix dev shell) so per-pack repos only contain pack-specific configuration and fixture data.

This template is built around two existing references:

- **Layout & convention**: based on the structural pattern from [`VisiCore/cc-edge-claude-code-io`](https://github.com/VisiCore/cc-edge-claude-code-io).
- **Test pattern**: TypeScript port of the [criblpacks](https://github.com/criblpacks) approach. The original Python test harness lives at [`criblpacks/cribl-palo-alto-networks/test/`](https://github.com/criblpacks/cribl-palo-alto-networks/tree/main/test); we ported it to Vitest per the [dryvist TypeScript-everywhere policy](https://github.com/dryvist/.github/blob/main/CLAUDE.md).

CI calls this repo's reusable workflows (`uses: dryvist/cc-edge-pack-template/.github/workflows/...`).

## Installation

Create a new pack repo from this template (admin token required for `gh repo create` against the `dryvist` org — see `~/git/org-dryvist/CLAUDE.md`):

```sh
gh repo create dryvist/cc-edge-mything-io \
  --template dryvist/cc-edge-pack-template \
  --public \
  --clone

cd cc-edge-mything-io
direnv allow              # activates nix-devenv typescript shell (node + pnpm + tsc + biome)
make install              # pnpm install in tests/
```

If you prefer the GitHub UI: navigate to this repo, click **Use this template** → **Create a new repository**.

## Usage

After scaffolding from the template:

1. **Customize `package.json`**: replace `name`, `description`, `displayName`, `tags`. Pack name MUST follow the validator convention `cc-edge-<source>-io` or `cc-stream-<source>-io`.
2. **Set the pack type in `.github/workflows/test.yml`**: change `pack_type: edge` to `stream` if this is a Stream pack.
3. **Define your inputs** in `default/inputs.yml`. Every input must declare `metadata.datatype` so route filters can match.
4. **Define your routes** in `default/pipelines/route.yml`. Replace the `REPLACE_*` placeholders. All routes MUST `output: __group` (validator rule).
5. **Define your pipelines** in `default/pipelines/<name>/conf.yml`. No pipeline named `main`. Eval functions should set `sourcetype` + `index` so the required-fields assertion passes (or place `tests/fixtures/.skip-required-fields` to opt out).
6. **Drop sample events** in `data/samples/*.json` and catalog them in `default/samples.yml`.
7. **Author test fixtures** in `tests/fixtures/<pipeline-name>/`:
   - `<case>.json` (input)
   - `<case>.expected.json` (optional partial-match expected output)
8. **Run locally**: `make docker-up && make test`
9. **Validate**: `make validate` builds the `.crbl` and prints the command to run [`/validate-pack`](https://github.com/VisiCore/vct-cribl-pack-validator) against it.
10. **Release**: do not tag manually — release-please proposes versions via PR. Merge the release-please PR to publish.

## Layout

```
.
├── .github/workflows/
│   ├── cribl-pack-test.yml      # GENERIC reusable — consumed by other dryvist packs
│   ├── cribl-pack-release.yml   # GENERIC reusable — consumed by other dryvist packs
│   ├── test.yml                 # CALLER — change pack_type per pack
│   ├── release.yml              # CALLER
│   └── release-please.yml       # CALLER — inherits JacobPEvans/.github
├── data/
│   └── samples/                 # PACK-SPECIFIC sample events
├── default/
│   ├── inputs.yml               # PACK-SPECIFIC source definitions
│   ├── pack.yml                 # PACK-SPECIFIC branding (logo)
│   ├── pipelines/
│   │   ├── route.yml            # PACK-SPECIFIC routes
│   │   └── <name>/conf.yml      # PACK-SPECIFIC pipeline functions
│   └── samples.yml              # PACK-SPECIFIC sample catalog
├── tests/
│   ├── cribl-client.ts          # GENERIC — TypeScript Cribl management API client
│   ├── parse-filter.ts          # GENERIC
│   ├── global-setup.ts          # GENERIC — Vitest globalSetup
│   ├── test-helpers.ts          # GENERIC
│   ├── routes.test.ts           # GENERIC — structural + dynamic flow tests
│   ├── pipelines.test.ts        # GENERIC — fixture-parametrized pipeline tests
│   ├── package.json             # GENERIC — pnpm workspace for tests
│   ├── tsconfig.json            # GENERIC
│   ├── vitest.config.ts         # GENERIC
│   └── fixtures/                # PACK-SPECIFIC test data
│       └── <pipeline>/
│           ├── <case>.json
│           └── <case>.expected.json
├── biome.jsonc                  # GENERIC — mirror of dryvist/.github
├── docker-compose.yml           # GENERIC
├── flake.nix                    # GENERIC — delegates to JacobPEvans/nix-devenv typescript shell
├── .envrc                       # GENERIC — `use flake`
├── Makefile                     # GENERIC
├── package.json                 # PACK-SPECIFIC — Cribl pack metadata
├── README.md                    # PACK-SPECIFIC
├── LICENSE                      # GENERIC — Apache-2.0
└── CLAUDE.md                    # GENERIC — AI assistant guidance
```

When the template improves, downstream packs should pull the GENERIC files via cherry-pick (or Renovate handles `biome.jsonc` automatically).

## API

This template doesn't expose a programmatic API. It provides:

- **CLI surface (Makefile)**: `make help`, `install`, `build`, `docker-up`, `docker-down`, `test`, `typecheck`, `lint`, `format`, `validate`, `clean`
- **pnpm scripts** (in `tests/`): `test`, `test:watch`, `typecheck`, `lint`, `format`
- **Test fixture surface**: filesystem convention under `tests/fixtures/<pipeline>/<case>.{json,expected.json}`
- **CI surface**: `cribl-pack-test.yml` + `cribl-pack-release.yml` reusable workflows callable from any dryvist pack via `uses: dryvist/cc-edge-pack-template/.github/workflows/...`

## Contributing

This template is the source of truth for shared pack infrastructure. Changes here propagate to every downstream pack.

When updating:

1. Make changes on a feature branch.
2. Open a PR against `main`. Note that the template's own CI workflows are gated on `is_template == false`, so they won't run here — verify against a real pack instead.
3. Pick a consumer pack (e.g. `dryvist/cc-edge-claude-code-io`) and apply the same changes there in a parallel PR. Confirm CI green.
4. Merge both.

Conventional commits required (`feat:`, `fix:`, `chore:`, `docs:`) — release-please uses these to compute version bumps.

## License

Apache-2.0 — see `LICENSE`.

## References

- [`dryvist/.github`](https://github.com/dryvist/.github) — org-wide standards (Biome config, security policy, AI assistant policy)
- [`VisiCore/cc-edge-claude-code-io`](https://github.com/VisiCore/cc-edge-claude-code-io) — structural reference pack (Python era)
- [`VisiCore/vct-cribl-pack-validator`](https://github.com/VisiCore/vct-cribl-pack-validator) — Claude Code skill running 27+ structural checks
- [`criblpacks`](https://github.com/criblpacks) — Cribl's official pack org; this template's test pattern is a TypeScript port of theirs
- [`JacobPEvans/nix-devenv`](https://github.com/JacobPEvans/nix-devenv) — provides the `typescript` dev shell (node, pnpm, tsc, biome) that `flake.nix` delegates to
- [Cribl management API](https://docs.cribl.io/api-reference/)
- [Cribl pack docs](https://docs.cribl.io/stream/packs/)
