# New pack from this template

End-to-end runbook for creating a new Cribl pack repo from
`dryvist/cc-edge-pack-template`. Six steps, ~15 minutes.

## 1. Create the repo

Repo creation in the dryvist org requires the org-admin PAT (the day-to-day
`GH_PAT_DRYVIST` cannot create repos). One-off wrapper:

```sh
GH_TOKEN=$(security find-generic-password -s GH_PAT_ORG_ADMIN -a ai-cli-coder \
  -w ~/Library/Keychains/elevate-access.keychain-db) \
  gh repo create dryvist/cc-edge-<source>-io \
    --template dryvist/cc-edge-pack-template \
    --public --clone
```

Naming convention enforced by `scripts/validate-pack-structure.sh`:
`cc-{edge,stream}-<source>-io`. The validator emits a warning (not an error)
for non-matching names, so deviation is possible but discouraged.

## 2. Install deps + git hooks

```sh
cd cc-edge-<source>-io
make install
```

Requires Node 20+ and pnpm 10+. Docker is only needed when you run `make test`.

## 3. Edit the manifest

`package.json` fields the validator hard-requires (`validate-pack-structure.sh`
will fail CI if any is missing or blank):

- `name` — e.g. `cc-edge-myservice-io`
- `version` — release-please rewrites this; the template ships `0.0.1`
- `minLogStreamVersion` — minimum Cribl version your pipeline relies on
  (e.g. `4.10.0`). See your Cribl release notes for the feature you depend on.

Soft-recommended (not validator-enforced, but used by the Cribl UI):
`displayName`, `description`, `tags`, `author`.

If this is a Cribl Stream pack (not Edge), flip
`.github/workflows/test.yml`: `pack_type: stream`.

## 4. Write your pipeline + first fixture

Replace the demo passthrough with the real pipeline:

- Pipeline config: `default/pipelines/<your-pipeline>/conf.yml`
- Route filter: edit `default/pipelines/route.yml` so its filter matches your
  expected input shape and points at `<your-pipeline>`
- Remove the demo: delete `default/pipelines/passthrough/` and
  `tests/fixtures/passthrough/`

Create the first fixture pair (the harness auto-discovers — no test code
edits needed):

```sh
mkdir -p tests/fixtures/<your-pipeline>
# Paste a real input event:
${EDITOR} tests/fixtures/<your-pipeline>/baseline.json
# Paste the desired output (partial-match: only fields you assert on):
${EDITOR} tests/fixtures/<your-pipeline>/baseline.expected.json
```

See [`test-harness.md`](test-harness.md) for the fixture convention.

## 5. Run tests locally

```sh
make docker-up && make test
```

`make test` runs `pnpm vitest` against the live Cribl container. All three test
files run: `routes.test.ts`, `pipelines.test.ts` (your fixtures),
`harness-teeth.test.ts` and `tarball-parity.test.ts` (template-level guards).

## 6. Push + let CI close it out

```sh
git checkout -b feat/initial-pack
git add . && git commit -m "feat: initial pipeline + fixture"
git push -u origin feat/initial-pack
gh pr create --fill
```

CI runs validate + the multi-version Cribl matrix (`latest` required; older
majors best-effort per [`test-harness.md`](test-harness.md)). Once merged,
release-please opens a release PR; merging that publishes the `.crbl` to
GitHub Releases.

## Cribl version matrix overrides

The reusable test workflow defaults to testing against `cribl/cribl:latest`
(required) plus the most recent previous-major patch (best-effort). If your
pack needs to pin or extend, override in your `.github/workflows/test.yml`:

```yaml
uses: dryvist/cc-edge-pack-template/.github/workflows/cribl-pack-test.yml@main
with:
  pack_type: edge
  cribl_versions: '[{"version":"latest","required":true},{"version":"4.17.1","required":true}]'
```

## Branch protection (one-time, per repo)

After the first green CI run, require the `Test pack pipelines (Cribl latest)`
status check in the repo's branch protection rules. Older-version legs stay
visible but non-blocking.
