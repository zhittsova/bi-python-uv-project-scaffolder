# scaffold.sh — Python Data-Science Project Scaffolder

A single Bash script that bootstraps a fully configured, production-ready
Python data-science / analytics workspace in one command.

```sh
./scaffold.sh my-project-name
```

No cookiecutter, no copier, no templates repo — just `bash` + `uv`.

---

## Table of Contents

- [Why](#why)
- [Prerequisites](#prerequisites)
- [Usage](#usage)
- [What gets created](#what-gets-created)
- [Generated files in detail](#generated-files-in-detail)
  - [Directory tree](#directory-tree)
  - [.gitignore](#gitignore)
  - [pyproject.toml](#pyprojecttoml)
  - [.pre-commit-config.yaml](#pre-commit-configyaml)
  - [Makefile](#makefile)
  - [CI workflow](#ci-workflow)
  - [VS Code configuration](#vs-code-configuration)
  - [CITATION.cff](#citationcff)
  - [Documentation templates](#documentation-templates)
  - [Binder](#binder)
  - [README.md & DEVELOP.md](#readmemd--developmd)
  - [main.py](#mainpy)
  - [Placeholder test](#placeholder-test)
- [First steps after scaffolding](#first-steps-after-scaffolding)
- [Customisation points](#customisation-points)
- [Distributing the scaffolder](#distributing-the-scaffolder)
- [FAQ](#faq)

---

## Why

Setting up a Python project with proper tooling (linter, formatter, type checker,
CI, dependency groups, workspaces, VS Code integration) takes hours of
copy-paste-tweak from old projects. This script captures every boilerplate file in
one place so you start writing code in minutes, not hours.

The choices baked in are opinionated but grounded in current best practices:

| Concern | Tool | Why |
|---|---|---|
| Package/env manager | [uv](https://docs.astral.sh/uv/) | Fast, deterministic, replaces pip + venv + pip-tools |
| Monorepo layout | uv workspaces | `packages/` + `apps/` with automatic member discovery |
| Build backend | `uv_build` | Lighter than setuptools/hatch, native src-layout support |
| Lint + format | [ruff](https://docs.astral.sh/ruff/) | One tool, one config, sub-second on large codebases |
| Type checking | [mypy](https://mypy.readthedocs.io/) strict | Catches bugs before runtime; Pylance supplements in VS Code |
| Pre-commit | [ruff hooks](https://github.com/astral-sh/ruff-pre-commit) + [pre-commit-hooks](https://github.com/pre-commit/pre-commit-hooks) | Consistent code quality on every commit |
| CI | GitHub Actions | Lint → typecheck → test on push/PR |
| Dependency groups | PEP 735 | Separate dev / notebooks deps without polluting production |

---

## Prerequisites

| Requirement | Install |
|---|---|
| **uv** ≥ 0.5 | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| **git** | Pre-installed on macOS / Linux; `brew install git` otherwise |
| **Bash** ≥ 4 | macOS ships zsh as default, but Bash is available at `/bin/bash` |

Optional (fetched at runtime):

- **Internet access** — to download GitHub's Python `.gitignore`. If offline, the
  script falls back to a minimal built-in version.

---

## Usage

```sh
# Clone or download scaffold.sh, then:
chmod +x scaffold.sh
./scaffold.sh my-cool-project
```

The script:
1. Creates `my-cool-project/` with the full directory tree
2. Fetches `.gitignore` from GitHub's Python template
3. Runs `uv init` to initialise the workspace root
4. Generates all config files (see below)
5. Runs `uv sync` to create the venv and install dev deps

When it finishes you'll see:

```
✅  my-cool-project scaffolded successfully.

Next steps:
  cd my-cool-project
  make setup          # install pre-commit hooks
  code my-cool-project.code-workspace

To add your first package:
  uv init --package packages/my_package --build-backend uv
  uv sync --all-packages

Push to GitHub:
  git add .
  git commit -m "first commit"
  git branch -M main
  git remote add origin https://github.com/yourname/my-cool-project.git
  git push -u origin main

See DEVELOP.md for the full developer guide.
```

The GitHub URL uses your `git config github.user` (or `$USER` as fallback).

### Arguments

| Arg | Description |
|---|---|
| `<project-name>` | The directory name and project name. Also used for the `.code-workspace` file, `CITATION.cff`, `README.md`, and `pyproject.toml`. |

The script reads `git config user.name` to populate the `CITATION.cff` author fields
and `git config github.user` for the GitHub username (used in the CI badge URL and
push instructions). If `github.user` isn't configured, it falls back to `$USER`.

> **Tip**: Set your GitHub username once with `git config --global github.user yourname`
> so the scaffolder always picks it up.

---

## What gets created

```
my-cool-project/
├── .github/
│   └── workflows/
│       └── ci.yml                  # GitHub Actions: lint → typecheck → test
├── .vscode/
│   └── settings.json               # Python, Pylance, ruff settings
├── apps/                           # Runnable apps / CLI entry points
├── data/
│   ├── external/.gitkeep
│   ├── interim/.gitkeep
│   ├── processed/.gitkeep
│   └── raw/.gitkeep
├── docs/
│   ├── ASSUMPTIONS.md
│   ├── DATA_GENERATION.md
│   ├── DATA_SOURCES.md
│   ├── DECISIONS_MEMO_TEMPLATE.md
│   ├── LIMITATIONS.md
│   ├── METHODS.md
│   └── METRIC_DEFINITIONS.md
├── notebooks/                      # Jupyter exploration
├── outputs/
│   ├── figures/.gitkeep
│   └── map_layers/.gitkeep
├── packages/                       # Reusable library code (src layout)
├── tests/
│   ├── __init__.py
│   └── test_placeholder.py
├── binder/
│   ├── postBuild                   # Installs deps via uv into Binder's env
│   └── runtime.txt                 # Pins Python version for Binder
├── .gitignore                      # GitHub Python template + project extras
├── .pre-commit-config.yaml         # ruff + pre-commit-hooks
├── CITATION.cff                    # Auto-populated from git config
├── DEVELOP.md                      # Developer guide (uv, deps, conventions)
├── main.py                         # Entry-point placeholder
├── Makefile                        # setup, sync, test, lint, format, etc.
├── my-cool-project.code-workspace  # VS Code multi-root workspace
├── pyproject.toml                  # Root config: deps, tools, workspace
├── pyrightconfig.json              # Pylance / pyright strict mode
└── README.md                       # Project readme with CI + Binder badges
```

---

## Generated files in detail

### Directory tree

| Directory | Purpose | Gitignored? |
|---|---|---|
| `packages/` | Library code — each sub-dir is a uv workspace member with src layout | No |
| `apps/` | Runnable apps / CLIs — each sub-dir is a workspace member | No |
| `notebooks/` | Jupyter notebooks (exploratory, not type-checked) | No |
| `docs/` | Methodology, metrics, assumptions, limitations, data sources | No |
| `data/{raw,interim,processed,external}/` | Data files at various processing stages | **Yes** (`.gitkeep` preserved) |
| `outputs/{figures,map_layers}/` | Generated artifacts | **Yes** (`.gitkeep` preserved) |
| `tests/` | Root-level test directory | No |
| `.vscode/` | Editor settings (shared via git) | No |
| `.github/workflows/` | CI pipeline | No |
| `binder/` | Binder config (`postBuild` + `runtime.txt`) for mybinder.org | No |

### .gitignore

Fetched live from
[github/gitignore/Python.gitignore](https://github.com/github/gitignore/blob/main/Python.gitignore)
— the same file GitHub uses when you select "Python" during repo creation. This ensures
coverage of `__pycache__/`, `.venv/`, `*.egg-info/`, `.mypy_cache/`, etc.

Project-specific extras are appended:
- OS junk files (`.DS_Store`, `Thumbs.db`, `.directory`, etc.)
- `.history/` (VS Code local history)
- `**/outputs/` (regenerated artifacts)

**Offline fallback**: if the `curl` fails, a minimal built-in `.gitignore` with the
essential Python entries is written instead.

### pyproject.toml

The root `pyproject.toml` serves as the **workspace root** (not an installable package).
Key sections:

| Section | What it configures |
|---|---|
| `[project]` | Name, version, Python ≥ 3.13 |
| `[dependency-groups]` | `dev` (mypy, pytest, ruff, pre-commit, pytest-cov) and `notebooks` (ipykernel, pandas, numpy, matplotlib, seaborn, plotly, requests, nbformat) |
| `[build-system]` | `uv_build` backend |
| `[tool.uv]` | `package = false`, `default-groups = ["dev"]` |
| `[tool.uv.workspace]` | `members = ["packages/*", "apps/*"]` |
| `[tool.pytest.ini_options]` | testpaths, strict markers, `-ra` |
| `[tool.ruff]` | `line-length = 120`, `target-version = "py313"`, select rules |
| `[tool.ruff.format]` | Double quotes, space indent |
| `[tool.mypy]` | Strict mode, all strict flags enabled |
| `[tool.pylint.format]` | `max-line-length = 120` (in case pylint is used) |

### .pre-commit-config.yaml

Two hook repos:

1. **ruff-pre-commit** (`v0.7.4`) — `ruff` (lint with `--fix`) and `ruff-format`
2. **pre-commit-hooks** (`v5.0.0`) — `check-toml`, `check-yaml`, `end-of-file-fixer`,
   `trailing-whitespace`

### Makefile

Generic targets that work for any project:

| Target | Command | Description |
|---|---|---|
| `make setup` | `uv sync` + `pre-commit install` | First-time setup |
| `make sync` | `uv sync --all-packages --all-extras --all-groups` | Re-sync after dep changes |
| `make test` | `uv run pytest -q` | Run tests |
| `make lint` | `uv run ruff check . --fix` | Lint with auto-fix |
| `make format` | `uv run ruff format .` | Format code |
| `make typecheck` | `uv run mypy` | Type check |
| `make check` | format + lint + typecheck + test | Full quality gate |
| `make precommit` | `uv run pre-commit run --all-files` | Run all pre-commit hooks |
| `make clean` | Remove caches, reset outputs | Clean build artifacts |

### CI workflow

`.github/workflows/ci.yml` runs on push to `main` and on all PRs:

1. Checkout → setup Python 3.13 → setup uv (`astral-sh/setup-uv@v4`)
2. `make sync` → `make lint` → `make typecheck` → `make test`

### VS Code configuration

**`.vscode/settings.json`** — per-folder settings:
- Python interpreter → `.venv/bin/python`
- pytest enabled, unittest disabled
- Pylance diagnostic overrides (suppress missing stubs noise)
- ruff native server enabled, format-on-save, organise imports on save
- Line length 120

**`pyrightconfig.json`** — Pylance / pyright:
- Strict type checking mode
- Include/extra paths for `packages/**/src`, `apps/**/src`, and tests
- `reportMissingTypeStubs` and `reportUnknownMemberType` suppressed

**`<project>.code-workspace`** — VS Code multi-root workspace file:
- Single root folder (`.`)
- Mirrors the settings from `settings.json`
- Extension recommendations: Python, Pylance, ruff

### CITATION.cff

Auto-populated from `git config user.name`:
- `given-names` and `family-names` split from the full name
- `date-released` set to today's date
- License: GPL-3.0-or-later

### Documentation templates

Seven starter docs in `docs/`:

| File | Purpose |
|---|---|
| `METHODS.md` | Computation methodology, formulas, rationale |
| `METRIC_DEFINITIONS.md` | Metric name, formula, units, config references |
| `ASSUMPTIONS.md` | Working assumptions (living document) |
| `LIMITATIONS.md` | Known limitations, edge cases, failure modes |
| `DATA_SOURCES.md` | Data origin, format, update cadence, license |
| `DATA_GENERATION.md` | How synthetic / demo data is created |
| `DECISIONS_MEMO_TEMPLATE.md` | Template for decision memos (context → findings → actions → caveats) |

Each file contains a heading and HTML comment prompts — fill them in as the project evolves.

### Binder

`binder/` contains two files that let anyone launch notebooks on
[mybinder.org](https://mybinder.org) without installing anything locally:

- **`runtime.txt`** — pins the Python version (e.g., `python-3.13`)
- **`postBuild`** — bootstraps `uv`, exports the `notebooks` dependency group from
  `pyproject.toml`, and installs any workspace packages found under `packages/`.
  This avoids duplicating dependencies in a separate `requirements.txt`.

Because the root `pyproject.toml` uses `uv_build` with `package = false` (virtual
workspace), repo2docker would fail trying to `pip install .`. Placing the config in
`binder/` makes repo2docker ignore the root `pyproject.toml` entirely.

> **Note:** mybinder.org has limited ephemeral disk space. Notebooks that write large
> files at runtime may hit the quota. For heavy pipelines, run locally with `make sync`.

### README.md & DEVELOP.md

**README.md** includes:
- CI + Binder badges (auto-populated with your GitHub username from `git config github.user`)
- Quickstart (`make setup`, `make check`)
- Project layout overview
- Links to all docs
- "About the scaffolding" section with links to uv/ruff/mypy docs
- Link to `DEVELOP.md`

**DEVELOP.md** is a comprehensive developer guide covering:
- Initial setup
- Adding workspace packages (`uv init --package --build-backend uv`)
- Dependency groups (PEP 735) with table
- Running things via `uv run` / Makefile
- CLI entry points
- Notebooks workflow
- Pre-commit & CI
- Project conventions (src layout, line length, quotes, strict typing)
- Creating a GitHub template repo

### main.py

A minimal entry-point placeholder:

```python
"""Project entry-point — wire up CLI or ad-hoc scripts here."""

def main() -> None:
    print("Hello from <project>!")

if __name__ == "__main__":
    main()
```

### Placeholder test

`tests/test_placeholder.py` — a single `assert True` test that proves the test runner
works. Replace it with real tests as you build out the project.

---

## First steps after scaffolding

```sh
cd my-cool-project

# 1. Install pre-commit hooks
make setup

# 2. Open in VS Code
code my-cool-project.code-workspace

# 3. Push to GitHub (commands shown by the scaffolder)
git add .
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/yourname/my-cool-project.git
git push -u origin main

# 4. Create your first library package
uv init --package packages/my_lib --build-backend uv
uv sync --all-packages

# 5. Create a CLI app
uv init --package apps/my_cli --build-backend uv
uv add --package my_cli my_lib    # depend on the library
uv sync --all-packages

# 6. Run the full quality gate
make check
```

---

## Customisation points

The script is a single file — edit it directly to match your preferences.

| Want to change… | Where to look |
|---|---|
| Python version | `uv init --python` flag + `pyproject.toml` `requires-python` + `ci.yml` `python-version` |
| Ruff rules | `[tool.ruff.lint] select` in the pyproject heredoc |
| Line length | `line-length` in ruff, pylint, and `.vscode/settings.json` sections |
| Dependency groups | `[dependency-groups]` in the pyproject heredoc |
| Default packages in notebooks group | The `notebooks = [...]` array |
| License | `CITATION.cff` `license` field + add a LICENSE file |
| CI provider | Replace the `.github/workflows/ci.yml` heredoc |
| Remove mypy strict | Delete the `[tool.mypy]` block or change `strict = false` |
| Add Docker | Add a Dockerfile heredoc section |
| Remove VS Code config | Delete the `.vscode/` and `.code-workspace` sections |

---

## Distributing the scaffolder

### Option A: GitHub template repository

1. Run `./scaffold.sh my-template` to generate a clean project
2. Push it to GitHub
3. Go to **Settings → General → Template repository** ☑
4. Others click **"Use this template"** to create new repos from it

### Option B: Gist / standalone repo

Keep `scaffold.sh` in a gist or small repo. Run it remotely:

```sh
curl -fsSL https://raw.githubusercontent.com/YOU/ds-template/main/scaffold.sh \
  | bash -s my-new-project
```

### Option C: Shell alias

```sh
# In ~/.zshrc or ~/.bashrc:
alias ds-scaffold='bash <(curl -fsSL https://raw.githubusercontent.com/YOU/ds-template/main/scaffold.sh)'

# Usage:
ds-scaffold my-new-project
```

---

## FAQ

**Q: Why not use cookiecutter / copier?**\
A: They're great tools. This script exists because (a) it has zero dependencies beyond
`bash` and `uv`, (b) every generated file is visible in one place (~780 lines), and
(c) it's trivial to fork and edit. For larger teams or more complex templating
(conditionals, prompts), cookiecutter/copier would be better.

**Q: Can I run it on an existing project?**\
A: No — by design, it creates a new directory and refuses to overwrite. To retrofit an
existing project, read through the script and copy the sections you need.

**Q: What if I don't want the monorepo layout?**\
A: Remove the `packages/` and `apps/` dirs, the `[tool.uv.workspace]` section, and the
related `src` paths in ruff/pyright configs. Use a flat `src/` layout instead.

**Q: Why `uv_build` instead of setuptools?**\
A: `uv_build` is purpose-built for uv workspaces, handles `src/` layout natively, and
doesn't require `setup.cfg` / `setup.py`. See
[uv docs](https://docs.astral.sh/uv/concepts/build-backend/).

**Q: Why mypy strict?**\
A: Strict mode catches more bugs. It's easier to start strict and selectively relax
(`# type: ignore[code]`) than to retroactively enable strictness on a large codebase.
