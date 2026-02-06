# CLAUDE.md

## What is this repo?

rawpy-demosaic is a GPL-3.0-licensed fork of [rawpy](https://github.com/letmaik/rawpy). The sole purpose of this fork is to build and distribute wheels that include the GPL2 and GPL3 demosaic algorithm packs (AMaZe, VCD, Modified AHD, LMMSE, etc.) enabled by default.

**The core rawpy functionality must remain identical to upstream.** Do not modify the rawpy Python API, Cython bindings, or LibRaw wrapper behavior. All changes in this fork should be limited to:

- CI/CD pipeline (`.github/workflows/ci.yml` and `.github/scripts/`)
- Build configuration (`setup.py`, `pyproject.toml`) to enable GPL demosaic packs
- Packaging metadata (package name, license, URLs)
- Documentation references (README, DEVELOP.md)

## Key differences from upstream rawpy

- Package name: `rawpy-demosaic` (not `rawpy`)
- License: GPL-3.0-or-later (due to GPL demosaic packs)
- LibRaw built with `-DENABLE_DEMOSAIC_PACK_GPL2=ON` and `-DENABLE_DEMOSAIC_PACK_GPL3=ON`
- Demosaic pack sources live in `external/LibRaw-demosaic-pack-GPL2` and `external/LibRaw-demosaic-pack-GPL3` (git submodules)
- This fork does not host its own docs — API docs link to upstream: https://letmaik.github.io/rawpy/api/

## Project structure

- `rawpy/` — Python package (Cython extension wrapping LibRaw)
- `external/` — Git submodules: LibRaw, LibRaw-cmake, demosaic packs
- `.github/workflows/ci.yml` — CI: build wheels, test, publish to PyPI
- `.github/scripts/` — Platform-specific build and test scripts
- `setup.py` — Build configuration including LibRaw compilation with demosaic packs
- `pyproject.toml` — Package metadata
- `test/` — Tests
- `docs/` — Sphinx docs source (not published by this fork)

## CI pipeline

The CI workflow (`ci.yml`) has three jobs:

1. **build** — Builds wheels for Linux (x86_64, aarch64), macOS (arm64), and Windows (x86_64) across Python 3.9–3.14
2. **test** — Installs built wheels and runs tests on all platform/version combos
3. **publish-wheels** — Publishes to PyPI on tagged pushes (`v*`)

This fork does **not** build or publish docs. Docs references should always point to the upstream rawpy docs.

## Syncing with upstream

```bash
git remote add upstream https://github.com/letmaik/rawpy.git
git fetch upstream
git merge upstream/main
```

When merging, be careful to preserve the GPL demosaic pack enablement in `setup.py` and the build scripts, and keep the package name as `rawpy-demosaic`.
