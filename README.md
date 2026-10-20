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
