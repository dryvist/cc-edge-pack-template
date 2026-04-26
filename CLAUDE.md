# CLAUDE.md — guidance for AI assistants working in this repo

This file is read by Claude Code (and other AI assistants supporting `CLAUDE.md`) on every session. It encodes the guardrails for working in this template — and, by inheritance, in any pack scaffolded from it.

## Repository Type

This is a **template repository** for new Cribl Edge / Stream packs in the dryvist org. It is consumed via `gh repo create --template`. The files here become the starting point for every downstream pack.

For the org-wide policy this template inherits from, read [`dryvist/.github/CLAUDE.md`](https://github.com/dryvist/.github/blob/main/CLAUDE.md).

## Language + Tooling Baseline

Per dryvist org policy: **TypeScript everywhere we write code**. Cribl pack content (`default/`, `data/samples/`, `tests/fixtures/`) stays JSON/YAML.

| Concern | Tool |
|---|---|
| Runtime | Node.js LTS |
| Package manager | pnpm |
| Test runner | Vitest |
| Lint + format | Biome |
| Type check | `tsc --noEmit` |
| Local dev shell | `nix develop` (delegates to [`JacobPEvans/nix-devenv`](https://github.com/JacobPEvans/nix-devenv) `typescript` shell — provides node, pnpm, tsc, biome, typescript-language-server) |
| Cribl runtime | `docker compose up -d` (Stream container; supports both Edge + Stream packs) |

`flake.nix` + `.envrc` wire the dev shell. `direnv allow` once per worktree, then everything's on PATH.

## Generic vs Pack-Specific

The single most important rule: **distinguish generic files from pack-specific files.**

**Generic** (DO NOT modify in pack repos — only in this template):

- `tests/cribl-client.ts`
- `tests/parse-filter.ts`
- `tests/global-setup.ts`
- `tests/test-helpers.ts`
- `tests/routes.test.ts`
- `tests/pipelines.test.ts`
- `tests/package.json`, `tests/tsconfig.json`, `tests/vitest.config.ts`
- `biome.jsonc` (mirror of `dryvist/.github/biome.jsonc`)
- `flake.nix`, `.envrc`
- `Makefile`
- `docker-compose.yml`
- `.github/workflows/cribl-pack-test.yml` (reusable)
- `.github/workflows/cribl-pack-release.yml` (reusable)
- `.github/workflows/test.yml` (caller — only `pack_type:` value changes per pack)
- `.github/workflows/release.yml` (caller)
- `.github/workflows/release-please.yml` (caller — inherits JacobPEvans pipeline)

If you find yourself wanting to modify any of the above in a pack repo, **stop**. Either:

1. The change belongs in this template (open a PR here, then propagate to packs), or
2. The change should be expressed as fixture data (`tests/fixtures/<pipeline>/`).

**Pack-specific** (free to modify per-pack):

- `package.json` (name, version, displayName, tags, description)
- `default/pack.yml` (logo)
- `default/inputs.yml` (sources)
- `default/pipelines/route.yml` (routes)
- `default/pipelines/<name>/conf.yml` (pipeline functions — Eval, Drop, etc.)
- `default/samples.yml` (sample catalog)
- `data/samples/*.json` (sample events)
- `tests/fixtures/<pipeline>/*.json` (test fixtures)
- `tests/fixtures/.skip-required-fields` (optional marker — opts out of required-fields assertion when sourcetype/index are set downstream rather than in-pack)
- `.release-please-manifest.json` (per-pack version state)
- `release-please-config.json` (per-pack release config)
- `README.md` (describe your specific pack)
- `LICENSE` (use Apache-2.0 unless instructed otherwise)

## Validator Rules (vct-cribl-pack-validator)

Always enforce these — they are non-negotiable per the [validator skill](https://github.com/VisiCore/vct-cribl-pack-validator):

| Rule | What it means |
|---|---|
| Pack ID format | `cc-edge-<source>-io` for Edge, `cc-stream-<source>-io` for Stream |
| No pipeline named `main` | All pipelines must have descriptive names |
| All routes use `output: __group` | Never `input_id` (breaks on source rename) |
| All sources have `metadata.datatype` | So route filters can match |
| Filters must be dynamic | Never literal `false` / `0` |
| No hardcoded paths | Use environment variables (`$MY_LOG_PATH`) |
| No hardcoded credentials | Use Cribl secrets |
| PII fields masked | `email`, `username`, `*_id`, `src_ip`, etc. before destinations |

## Fixture Convention

When adding tests, follow filesystem convention — no TypeScript edits required:

```
tests/fixtures/<pipeline-name>/<case>.json           # input
tests/fixtures/<pipeline-name>/<case>.expected.json  # optional expected output (partial match)
```

The generic `pipelines.test.ts` auto-discovers and parametrizes one Vitest case per `<case>.json` it finds. If `<case>.expected.json` is missing, the case is a smoke test (asserts non-empty output only). When you add an expected file, the assertions tighten automatically. Required-fields assertion (`sourcetype` + `index` for Edge; `host` + `source` + `_time` for Stream) runs unconditionally unless `tests/fixtures/.skip-required-fields` exists.

Prefer richer fixtures over richer TypeScript. If the assertion can't be expressed as a partial-match expected event, it probably shouldn't be a test — consider whether it's really a pipeline behavior or something else.

## Don't Invent — Reuse

- **Use existing Cribl tooling** — `cribl pipe`, the management API, official Docker images. Don't reinvent.
- **Use existing third-party Actions** — `softprops/action-gh-release`, `rlespinasse/github-slug-action`, `actions/setup-node`, `pnpm/action-setup`. Don't write custom packaging shell scripts.
- **Use the criblpacks pattern** — that's where `cribl-client.ts` derives from (TypeScript port). When extending, mirror their idioms.
- **Use vct-cribl-pack-validator** — for deep structural validation, not custom-rolled YAML parsers.

## Workflow

For any pack work:

1. `/refresh-repo` then create a worktree for the change (per user's global `CLAUDE.md`).
2. `direnv allow` to activate the typescript dev shell.
3. Modify only pack-specific files (see lists above).
4. `make test` locally before committing.
5. `make validate` before tagging a release.
6. **Don't tag versions yourself** — release-please proposes them via PR; the human controls release timing.

## When in Doubt

- Read [`dryvist/.github/CLAUDE.md`](https://github.com/dryvist/.github/blob/main/CLAUDE.md) — org-wide policy.
- Read [`VisiCore/cc-edge-claude-code-io`](https://github.com/VisiCore/cc-edge-claude-code-io) — the gold-standard reference pack (Python-era; structure carries over).
- Read [`criblpacks/cribl-palo-alto-networks`](https://github.com/criblpacks/cribl-palo-alto-networks) — Cribl's own test pattern reference (Python).
- Read [`VisiCore/vct-cribl-pack-validator`](https://github.com/VisiCore/vct-cribl-pack-validator) — the authoritative ruleset.
- Don't add scripts. If you're tempted to write a script, ask first whether a Cribl-native or GitHub Action equivalent already exists.
