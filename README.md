# REALwrap

A Python client and CLI toolkit for the [Enamine REAL Tools API](https://real.enamine.net/api/docs) -
search REAL space for synthesis routes, extract building blocks and reaction centre SMARTS, and
prepare [Syndirella](https://github.com/oxpig/syndirella/) input files.

Originally developed as part of [XChem-CAR](https://github.com/waztom/xchem-car) and extracted into a standalone
package so it can be used without running the full CAR application.

## Installation

```bash
pip install REALwrap
```

Until the first PyPI release is published, install directly from GitHub:

```bash
pip install git+https://github.com/xchem/xchem-REALwrap.git
```

## Getting an API key

Obtain an API key from [real.enamine.net/api/docs](https://real.enamine.net/api/docs).

## Quick start

```python
from realwrap import RealWrapClient

client = RealWrapClient(api_key="your-api-key")

results = client.search_smiles_single("CNC(=O)C1Cc2ccccc2N1C(=O)c1cccc(OCC(F)(F)F)n1")
```

Unlike the original CAR-embedded module, the API key is passed explicitly when constructing the
client - it is never read from an environment variable.

## Command line tools

Three console scripts are installed alongside the package:

| Command | Purpose |
|---|---|
| `realwrap-fetch` | Search a CSV of target SMILES against REAL space and produce a processed output CSV |
| `realwrap-extract-bbs` | Extract deduplicated building blocks and reaction-centre SMARTS from a REAL Tools output CSV |
| `realwrap-prepare-syndirella` | Prepare a Syndirella manual input CSV + SMIRKS JSON from a Syndirella auto input |

All three require an explicit `--api-key` (except `realwrap-extract-bbs`, which only operates on an
existing CSV, and `realwrap-prepare-syndirella --enamine-csv ...` in offline mode). Run any command
with `--help` for full options.

```bash
realwrap-fetch targets.csv --api-key YOUR_KEY -o processed.csv
realwrap-extract-bbs enamine_output.csv building_blocks.csv
realwrap-prepare-syndirella syndirella_auto.csv --api-key YOUR_KEY -o manual.csv -j smirks.json
```

## Reaction types

| Code | Description |
|---|---|
| 0 | Very tractable |
| 1050 | Moderately tractable |
| 1051 | Legacy |
| 1055 | Custom synthesis |

## Development

This project uses [uv](https://docs.astral.sh/uv/) to create the virtual environment and install
dependencies, matching the tooling used in CI (see [.github/workflows/tests.yaml](.github/workflows/tests.yaml)
and [.github/workflows/release.yaml](.github/workflows/release.yaml)).

Install `uv` if you don't already have it (see the
[uv installation guide](https://docs.astral.sh/uv/getting-started/installation/) for other methods):

```bash
pip install uv
```

Create an isolated virtual environment, then install the package with its dev/test dependencies:

```bash
uv venv .venv
source .venv/bin/activate
uv pip install -e ".[dev]"
```

Run the test suite:

```bash
pytest tests/
```

### Manual SMIRKS regression test

[tests/test_smirks_regression.py](tests/test_smirks_regression.py) is a standalone diagnostic script, not
part of the automated `pytest` suite (it's excluded via `addopts` in `pyproject.toml`). For every `FOUND` row
in one or more `realwrap-fetch`-produced processed CSVs, it looks up the SMIRKS for that row's
`reaction_name`, desalts the reactant SMILES, applies the reaction with RDKit, and checks the result against
the row's `target_smiles`/`found_smiles`. It then reports per-reaction-type and grand-summary pass/fail
statistics, so it's useful for sanity-checking that `prepare_syndirella_input.py`'s SMIRKS mappings still
reproduce real REAL space chemistry after a change.

To run it:

1. Produce a processed CSV with `realwrap-fetch` (or reuse an existing one from a prior run).
2. Edit the `processed_files` list in `test_smirks_regression.py`'s `main()` to point at your CSV path(s) -
   it ships with placeholder paths from the original CAR container.
3. Run it directly:

   ```bash
   python tests/test_smirks_regression.py
   ```

It exits non-zero if the 2-component pass rate drops below 50%.

## Releasing

Tests run automatically on every push/PR to `main` and `staging` (see
[.github/workflows/tests.yaml](.github/workflows/tests.yaml)).

Publishing to PyPI is triggered by publishing a GitHub Release (see
[.github/workflows/release.yaml](.github/workflows/release.yaml)), using
[PyPI Trusted Publishing](https://docs.pypi.org/trusted-publishers/) (OIDC, no stored secrets).
The release/tag name must be a bare [PEP 440](https://peps.python.org/pep-0440/) version, e.g.
`0.2.0` (no `v` prefix), since it is passed directly to `uv version` to set the package version
before building - the version in `pyproject.toml` is not used.

## License

MIT
