# Test harness

Vitest-based, lives entirely in `tests/`. Auto-discovers fixtures by filesystem
convention — no TypeScript edits needed to add tests.

## What gets tested

| Suite | What it asserts |
|---|---|
| `routes.test.ts` (structure) | route.yml exists, every route has a pipeline, every referenced pipeline file exists, routes use `output: __group`, filters aren't statically falsy, no pipeline named `main` |
| `routes.test.ts` (dynamic flow) | Per route: a synthetic event matching its filter triggers the named pipeline and isn't dropped (uses live Cribl) |
| `pipelines.test.ts` | Per fixture: pipeline produces non-empty output; partial-match against `<case>.expected.json` if present; required-fields assertion (`sourcetype`+`index` for Edge; `host`+`source`+`_time` for Stream) unless `.skip-required-fields` marker present |
| `tarball-parity.test.ts` | The whitelist in `tests/cribl-client.ts::PACK_ROOT_ENTRIES` (used by every test-time pack install) matches `INCLUDE=` in `scripts/build-crbl.sh` (used by every release). Catches drift before a release ships a tarball CI never validated. |
| `harness-teeth.test.ts` | Meta-tests: every assertion helper used by the suites above actually throws on its target failure mode. Pure unit-level; no Cribl required. |

## Coverage matrix (what the harness proves it catches)

Every guard listed here has at least one teeth-test in `harness-teeth.test.ts`.
Adding a guard? Add a teeth-test in the same PR.

| Guard | Lives in | Catches | Proven by |
|---|---|---|---|
| `assertPartialMatch` — missing expected key | `tests/test-helpers.ts` | Pipeline drops a field the fixture expects | `harness-teeth.test.ts` → "throws when an expected key is missing" |
| `assertPartialMatch` — wrong value | `tests/test-helpers.ts` | Pipeline sets a field to the wrong value | `harness-teeth.test.ts` → "throws when an expected value differs" |
| `assertPartialMatch` — event-count mismatch | `tests/test-helpers.ts` | Pipeline duplicates/drops events | `harness-teeth.test.ts` → "throws on event-count mismatch (actual longer/shorter)" |
| Smoke assertion (`length > 0`) | `tests/pipelines.test.ts` (inline) | Pipeline drops every event under a real filter/eval | `harness-teeth.test.ts` covers the underlying `expect().toBeGreaterThan` indirectly via Vitest; observed in CI when a pipeline regression empties output |
| `assertRequiredFields` — missing edge field | `tests/cribl-client.ts` | Edge pipeline output lacks `sourcetype` or `index` | `harness-teeth.test.ts` → "throws when edge sourcetype/index is missing" |
| `assertRequiredFields` — missing stream field | `tests/cribl-client.ts` | Stream pipeline output lacks `host`/`source`/`_time` | `harness-teeth.test.ts` → "throws when stream host/_time is missing" |
| Tarball whitelist drift | `tests/tarball-parity.test.ts` | `build-crbl.sh` and `createPackTarball` diverge | `tarball-parity.test.ts` (also self-proving: flip an `INCLUDE=` entry and watch CI go red) |

If you add a new assertion helper, add a teeth-test in the same PR or coverage rots silently.

## Fixture convention

```text
tests/fixtures/<pipeline-name>/<case>.json           # input event(s)
tests/fixtures/<pipeline-name>/<case>.expected.json  # optional partial-match expected output
tests/fixtures/.skip-required-fields                 # optional org-wide opt-out marker
```

The generic `pipelines.test.ts` auto-discovers and parametrizes one Vitest case
per `<case>.json`. Add a fixture → tests run automatically. Remove a fixture →
tests stop running. No code changes.

## Cribl version matrix

The reusable test workflow (`cribl-pack-test.yml`) runs Vitest in parallel
against every Cribl version listed in its `cribl_versions` input. Single
source of truth: the input's JSON-array default.

Shape:

```json
[
  {"version": "latest", "required": true},
  {"version": "3.5.4",  "required": false}
]
```

- `required: true` — failure blocks PR merges (required status check)
- `required: false` — failure is visible but non-blocking (best-effort smoke)

Each leg posts a distinct GitHub status check named
`Test pack pipelines (Cribl <version>)`, so branch-protection rules can
require the `latest` check specifically while leaving older legs informational.

**Extending the matrix.** Add a JSON entry. No other edits. For example, to
also pin against the previous minor of the current major:

```yaml
# In .github/workflows/test.yml (the per-pack caller)
uses: dryvist/cc-edge-pack-template/.github/workflows/cribl-pack-test.yml@main
with:
  pack_type: edge
  cribl_versions: '[{"version":"latest","required":true},{"version":"4.17.1","required":false},{"version":"3.5.4","required":false}]'
```

**Version policy.**

- `latest` always floats — tracks the current major's newest patch (4.x today;
  will track 5.x once Cribl ships it).
- Pinned older entries should be exact patches (e.g. `3.5.4`, `4.17.1`) since
  EOL or older lines don't get new patches.
- When the current major changes, pin the last patch of the outgoing major as
  a new entry: `{"version": "4.18.1", "required": false}`.

## Required-fields assertion

Edge packs are expected to set `sourcetype` and `index` (typically via an Eval
function in the pipeline). Stream packs set `host`, `source`, `_time`. The
assertion fires unconditionally on every pipeline test unless you opt out by
creating an empty `tests/fixtures/.skip-required-fields` file (use this for
pass-through packs whose downstream sets these fields).

## CriblClient API surface

`tests/cribl-client.ts` wraps the Cribl management API. Reference:

| Method | Purpose |
|---|---|
| `waitUntilReady()` | Block until `/health` returns 200 |
| `installPack(tarball, expectedId?)` | Upload `.crbl` + poll until pack registers |
| `deletePack(packId)` | Remove pack |
| `saveSample(name, events)` / `deleteSample(id)` | Sample lifecycle |
| `runPipeline(pipeline, sampleId, {pack})` | Execute via `/preview` (mode `pipe`); returns output events |
| `runRouteFlow(sampleId, events, {pack})` | Local route-matcher fallback (Cribl has no `mode:route`); finds matching route in `route.yml`, then runs its pipeline |
| `assertRequiredFields(events, packType?)` | Assert canonical fields per pack type |
| `startCapture(filter, ...)` / `readCapture(id, ...)` | Live capture primitives (reserved for future integration tests) |
| `createPackTarball(packRoot)` (static) | Build `.crbl` from on-disk pack contents |
