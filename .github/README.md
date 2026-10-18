# `.github/` — workflow callers for Cribl packs

The workflows in this directory are event callers. The pack test and release
jobs call reusable workflows from `jpe-cribl/.github` at a pinned commit, which
provide the CI behavior and the shared scripts.

## Installation

These caller workflows are copied when a pack is scaffolded from the template
(`gh repo create --template`). The reusable workflows resolve at runtime from
`jpe-cribl/.github`; shared CI scripts are checked out under `.gh-shared/` in CI.

## Usage

- A PR touching `default/`, `data/samples/`, `tests/`, `scripts/`, or
  `package.json` runs `test.yml`.
- A version tag runs the `pack-release` job in `release.yml` and publishes the
  `.crbl` artifact.
- Version-bump automation is `_release.yml` from `JacobPEvans/.github`.

## Caller workflows in this repo

| File | Trigger | Calls |
| --- | --- | --- |
| `workflows/test.yml` | PR + push to main (paths-filtered) | `cribl-pack-test.yml` (`jpe-cribl/.github`) |
| `workflows/release.yml` | push to main + PR; semver tag push | `_release.yml` (`JacobPEvans/.github`) and `cribl-pack-release.yml` (`jpe-cribl/.github`) |

## Updating shared workflow inputs

Adding or renaming an input can affect every consumer pack. Keep input defaults
backward compatible whenever possible.

## Pinning external actions

Per [`SECURITY.md`](https://github.com/JacobPEvans/.github/blob/main/SECURITY.md):

- Trusted (`actions/*`, `pnpm/action-setup`):
  semver tag pins (`@v4`, etc.)
- Untrusted (everything else): SHA pins. Renovate (per the org's
  `renovate.json`) auto-converts version tags to SHA on first run.
