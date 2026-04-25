# Tests

Adopts the [criblpacks](https://github.com/criblpacks) test pattern (Python + Docker + Cribl management API) — generic and DRY across all packs.

## Installation

From the pack root, install Python test dependencies into a local virtualenv:

```sh
python3 -m venv tests/.venv
tests/.venv/bin/pip install -r tests/requirements.txt
```

Or via the Makefile (creates the same venv):

```sh
make install
```

Tests require Docker for the ephemeral `cribl/cribl` container — see `docker-compose.yml` at the pack root.

## Usage

From the pack root:

```sh
make docker-up    # start cribl/cribl Docker container
make test         # run pytest
make docker-down  # stop the container
```

Or run pytest directly (assuming the container is already running):

```sh
cd tests
.venv/bin/python -m pytest -v
```

In CI, the reusable workflow `dryvist/.github/.github/workflows/cribl-pack-test.yml` runs the same `pytest tests/` against an ephemeral `cribl/cribl` service container.

## Layout

```
tests/
├── conftest.py           # pytest fixtures (Cribl client, pack install/cleanup)
├── cribl_client.py       # API wrapper (adapted from criblpacks/cribl-palo-alto-networks)
├── test_pipelines.py     # GENERIC parametrized over fixtures/
├── test_routes.py        # GENERIC route.yml structural assertions
├── fixtures/
│   ├── <pipeline-name>/
│   │   ├── <case>.json              # input event(s)
│   │   └── <case>.expected.json     # (optional) expected output
│   └── ...
├── requirements.txt
└── README.md             # this file
```

## Fixture Convention

For each pipeline named `<pipeline-name>` (in `default/pipelines/<pipeline-name>/conf.yml`):

- `tests/fixtures/<pipeline-name>/<case>.json` — input event or list of events
- `tests/fixtures/<pipeline-name>/<case>.expected.json` — optional expected output (partial match)

Both files contain JSON. Inputs may be a single object or a list of objects (each with `_raw` if you have raw text, or any structured fields).

If `.expected.json` is missing, the test only asserts the pipeline produced non-empty output (smoke test).

## What's generic vs pack-specific

| File | Status |
|---|---|
| `cribl_client.py`, `conftest.py`, `test_pipelines.py`, `test_routes.py`, `requirements.txt` | **Generic** — copied unchanged from template, never modify per pack |
| `fixtures/<pipeline-name>/*.json` | **Per-pack** — author one input + one expected per pipeline behavior |

If you find yourself wanting to write pack-specific Python test code, first ask whether the assertion can be expressed as an `expected.json` partial-match. If yes, prefer that. If no (e.g. complex cross-event invariants), add a `tests/test_<packname>_extras.py` file alongside the generic ones.

## API

Test discovery is driven by filesystem convention — there is no Python API to call. The pytest collector walks `tests/fixtures/<pipeline-name>/` and parametrizes one test case per `<case>.json` it finds.

For lower-level scripting against a running Cribl instance, the `CriblClient` class in `cribl_client.py` exposes:

- `wait_until_ready(timeout_seconds)` — block until `/api/v1/health` responds
- `create_pack_tarball(pack_root)` / `install_pack(tarball)` / `delete_pack(pack_id)`
- `save_sample(name, events)` / `delete_sample(sample_id)` / `delete_all_samples()`
- `run_pipeline(pipeline, sample_id, pack)` — execute pipeline against saved sample

## Contributing

Generic test files (`cribl_client.py`, `conftest.py`, `test_pipelines.py`, `test_routes.py`, `requirements.txt`) live in `dryvist/cc-edge-pack-template`. Changes must be made there and propagated to consuming packs — do not modify in-place inside a pack repo without first updating the template.

## License

Apache-2.0 (matches the template).
