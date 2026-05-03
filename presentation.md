# Introduction to uv

**An extremely fast Python package & project manager, written in Rust**

*One tool to replace pip, pip-tools, pipx, poetry, pyenv, virtualenv, twine, and more — 10–100× faster than the originals.*

```
Python -> venv -> lock -> sync -> run -> publish
```

Resolve  |  Lock  |  Sync  |  Run

---

## Table of Contents

1. [Topics](#slide-01--topics)
2. [What Is uv?](#slide-02--what-is-uv)
3. [Why uv? — Speed & Consolidation](#slide-03--why-uv--speed--consolidation)
4. [Installation](#slide-04--installation)
5. [uv pip — Drop-In Replacement](#slide-05--uv-pip--drop-in-replacement)
6. [uv venv — Virtual Environments](#slide-06--uv-venv--virtual-environments)
7. [uv init — Starting a Project](#slide-07--uv-init--starting-a-project)
8. [pyproject.toml — Source of Truth](#slide-08--pyprojecttoml--source-of-truth)
9. [uv add / uv remove](#slide-09--uv-add--uv-remove)
10. [The Lockfile — uv.lock](#slide-10--the-lockfile--uvlock)
11. [uv sync — Reconcile the Environment](#slide-11--uv-sync--reconcile-the-environment)
12. [uv run — Execute in the Project Env](#slide-12--uv-run--execute-in-the-project-env)
13. [Managing Python Versions](#slide-13--managing-python-versions)
14. [PEP 723 — Inline Script Dependencies](#slide-14--pep-723--inline-script-dependencies)
15. [uvx & uv tool — Replacing pipx](#slide-15--uvx--uv-tool--replacing-pipx)
16. [Workspaces — Cargo for Python](#slide-16--workspaces--cargo-for-python)
17. [Dependency Groups & Optional Extras](#slide-17--dependency-groups--optional-extras)
18. [Build & Publish](#slide-18--build--publish)
19. [Performance Internals](#slide-19--performance-internals)
20. [uv vs pip / Poetry / pdm / Hatch](#slide-20--uv-vs-pip--poetry--pdm--hatch)
21. [Migrating from pip / requirements.txt](#slide-21--migrating-from-pip--requirementstxt)
22. [Migrating from Poetry](#slide-22--migrating-from-poetry)
23. [Configuration — pyproject.toml & uv.toml](#slide-23--configuration--pyprojecttoml--uvtoml)
24. [CI Patterns — GitHub Actions](#slide-24--ci-patterns--github-actions)
25. [Cheat Sheet — Daily Commands](#slide-25--cheat-sheet--daily-commands)
26. [Gotchas & Troubleshooting](#slide-26--gotchas--troubleshooting)
27. [Summary & Next Steps](#slide-27--summary--next-steps)

---

## Slide 01 — Topics

### Foundations

- What uv is — Astral, Rust, the consolidation pitch
- Installation and the first `uv pip` command
- Virtual environments with `uv venv`
- Starting a project with `uv init`

### Project workflow

- `pyproject.toml`, `uv add`, `uv remove`
- The universal `uv.lock` file
- `uv sync` — install & reconcile
- `uv run` — auto-sync & execute

### Power features

- Managing Python versions — `uv python`
- PEP 723 inline-script dependencies
- `uvx` for one-shot CLI tools
- Workspaces & dependency groups
- Build & publish to PyPI

### Adoption & performance

- Performance internals — cache, hardlinks, PubGrub
- pip / poetry / pdm / Hatch comparison
- Migration from `requirements.txt` / Poetry
- CI patterns and gotchas

---

## Slide 02 — What Is uv?

**uv** is a single Rust-based binary that handles every step of the Python packaging lifecycle — installing the interpreter, creating environments, resolving and locking dependencies, running scripts, building wheels, and publishing them.

### Origin & status

- Built by **Astral** — same team as `ruff`
- First released February 2024; 1.0 entered the project-manager era in late 2024
- Apache 2.0 / MIT dual-licensed, fully open source
- Single static Rust binary — no Python bootstrap required

### What uv replaces

- **pip** — package install / uninstall
- **pip-tools** — `pip-compile`, `pip-sync`
- **virtualenv** / `python -m venv`
- **pipx** — run / install standalone CLIs
- **pyenv** — install & switch CPython builds
- **poetry / pdm / Hatch** — project & lockfile workflow
- **twine / build** — build & publish to PyPI

The pitch: *"the Cargo of Python"* — one fast tool, one config file, one lockfile, no plugin sprawl.

---

## Slide 03 — Why uv? — Speed & Consolidation

### Speed (typical, cold cache → warm)

| Operation | pip | uv |
|---|---|---|
| Resolve `transformers` + extras | ~30 s | ~0.4 s |
| Install Django + tree | ~7 s | ~0.2 s |
| `poetry lock` on a medium project | ~25 s | ~0.3 s |
| Re-create venv from cache | ~3 s | ~50 ms |

Measured by Astral; varies by network. **10–100×** over pip is the rule, not the exception.

### Where the speed comes from

- **Rust** + zero-copy parsing + parallel HTTP
- **PubGrub** resolver with conflict-driven backtracking
- **Global content-addressed cache** shared across every project
- **Hardlinks / reflinks** from cache into venvs — install is essentially `ln`
- Aggressive use of HTTP range requests against PyPI's metadata index

### Beyond speed

- One binary, one config, one lockfile — drop-in for hostile CI containers
- Universal lockfile resolves for every platform / Python version at once
- Scripts can declare dependencies (**PEP 723**) — no project required

---

## Slide 04 — Installation

uv is distributed as a **single static binary**. No Python required — uv can install Python for you.

### Standalone installer

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

# pin a version
curl -LsSf https://astral.sh/uv/0.5.0/install.sh | sh
```

### Package managers

```bash
brew install uv          # macOS
winget install astral-sh.uv
pacman -S uv             # Arch
pipx install uv          # if pipx already there
pip install uv           # last resort
```

### Docker

```dockerfile
# Astral's slim image
FROM ghcr.io/astral-sh/uv:0.5-python3.12-bookworm-slim

# or copy the binary
COPY --from=ghcr.io/astral-sh/uv:latest /uv /uvx /usr/local/bin/
```

### Self-update

```bash
uv self update
uv self update --version 0.5.4
uv --version
```

### First sanity check

```text
$ uv --version
uv 0.5.4

$ uv python list
cpython-3.13.0-linux-x86_64-gnu
cpython-3.12.7-linux-x86_64-gnu
cpython-3.11.10-linux-x86_64-gnu
```

> **Tip** — install via the standalone installer in CI; it does not depend on a working Python. Pin the version in production: `UV_VERSION=0.5.4`.

---

## Slide 05 — uv pip — Drop-In Replacement

`uv pip` mirrors the pip CLI surface. **Faster, no behavioural changes** — switch in seconds for legacy projects.

### Same commands, faster

```bash
# classic pip
pip install requests
pip install -r requirements.txt
pip freeze > requirements.txt
pip install -e .
pip uninstall flask

# uv equivalent — same flags
uv pip install requests
uv pip install -r requirements.txt
uv pip freeze > requirements.txt
uv pip install -e .
uv pip uninstall flask
```

### Compile & sync (replaces pip-tools)

```bash
# produce a fully-pinned requirements file
uv pip compile requirements.in -o requirements.txt

# sync the venv to exactly match
uv pip sync requirements.txt

# add hashes for supply-chain integrity
uv pip compile requirements.in --generate-hashes -o requirements.txt
```

### When to use `uv pip` vs `uv add`

- **uv pip** = imperative, ad-hoc, no `pyproject.toml` required — perfect for legacy `requirements.txt` repos and one-off venvs.
- **uv add** = declarative project workflow — edits `pyproject.toml` and updates `uv.lock`.
- Use `uv pip` as the on-ramp. Migrate to `uv add` when you want a lockfile.

### Useful pip-mode flags

```bash
uv pip install \
  --index-url https://pypi.org/simple \
  --extra-index-url https://internal/index \
  --python /usr/bin/python3.11 \
  --system        \
  --no-cache      \
  --offline       \
  package==1.2.*
```

---

## Slide 06 — uv venv — Virtual Environments

### Create & activate

```bash
# defaults to .venv
uv venv

# pick a version
uv venv --python 3.12
uv venv --python 3.12.4
uv venv --python pypy@3.10

# pick a path
uv venv ~/envs/scratch

# activate (still POSIX as ever)
source .venv/bin/activate
.\.venv\Scripts\activate    # Windows
```

### You usually don't need to activate

Every `uv pip`, `uv run`, `uv add` auto-discovers `.venv` in the current directory. The only reason to `source` is for ad-hoc `python` / `pytest` calls in your shell.

### How fast is venv creation?

- `python -m venv`: ~3–5 s (copies CPython, runs ensurepip)
- `uv venv`: **~50 ms** — uses managed Python and skips ensurepip
- Throwaway venvs become genuinely cheap — useful in tests, hooks, ephemeral CI containers

### Seed packages

```bash
# by default the venv has no pip!
uv venv

# add the legacy seed if you really need it
uv venv --seed   # installs pip, setuptools, wheel
```

uv installs straight into `site-packages`, so pip-in-venv is mostly vestigial. Only enable seeding when external tools insist on calling `pip`.

---

## Slide 07 — uv init — Starting a Project

### Create a new project

```text
$ uv init my-app
Initialized project `my-app` at /tmp/my-app

$ tree -a my-app
my-app/
├── .gitignore
├── .python-version       # pinned Python (e.g. 3.12)
├── README.md
├── main.py               # hello-world entry
└── pyproject.toml
```

```bash
# init in-place
uv init --name my-app .

# library scaffold (with src/)
uv init --lib mylib

# application scaffold (default)
uv init --app my-cli

# packaged (build-system + entry points)
uv init --package my-tool
```

### Scaffold types

- **--app** (default) — flat layout, no build system
- **--lib** — `src/` layout, Hatch build system, ready for PyPI
- **--package** — application that should be pip-installable, with console scripts

### First commands

```bash
cd my-app
uv add requests           # creates .venv + uv.lock
uv run main.py            # runs in synced env
uv run python -V          # 3.12.x
```

### `.python-version`

A plain text file naming the required Python. Honoured by uv, pyenv, and most IDEs. Commit it.

---

## Slide 08 — pyproject.toml — Source of Truth

### Anatomy of a uv-managed project

```toml
[project]
name = "my-app"
version = "0.1.0"
description = "Example service"
readme = "README.md"
requires-python = ">=3.11"
authors = [
  { name = "Brendan Lynskey", email = "you@example.com" }
]
dependencies = [
  "fastapi>=0.115",
  "httpx>=0.27",
  "pydantic>=2.8",
]

[project.optional-dependencies]
postgres = ["asyncpg>=0.29"]

[dependency-groups]
dev = [
  "pytest>=8",
  "ruff>=0.6",
  "mypy>=1.10",
]

[tool.uv]
package = true            # install in editable mode
```

### `dependencies` vs groups vs extras

- **dependencies** — runtime, always installed
- **[project.optional-dependencies]** — extras users opt into when installing your package: `pip install my-app[postgres]`
- **[dependency-groups]** — dev-only groups (PEP 735). Never published, only synced locally / in CI.

### Version specifiers — quick reference

| Spec | Means |
|---|---|
| `requests` | any version |
| `requests==2.31` | exactly |
| `requests~=2.31` | compatible (≥ 2.31, < 3.0) |
| `requests>=2.0,<3` | range |
| `requests; sys_platform != "win32"` | marker |

---

## Slide 09 — uv add / uv remove

The everyday verbs. Each call edits `pyproject.toml`, refreshes `uv.lock`, and syncs the venv — atomically.

### Adding

```bash
# runtime deps
uv add fastapi httpx

# version pin
uv add 'pydantic>=2.8,<3'

# dev-group dependency
uv add --group dev pytest ruff mypy

# extras (optional dep)
uv add --optional postgres asyncpg

# from a local path / git / URL
uv add ./libs/internal-utils
uv add 'git+https://github.com/foo/bar@main'
uv add 'https://files.pythonhosted.org/packages/.../wheel.whl'

# editable
uv add --editable ./libs/internal-utils
```

### Removing & upgrading

```bash
# remove
uv remove httpx
uv remove --group dev pytest

# upgrade everything within constraints
uv lock --upgrade

# upgrade just one package
uv lock --upgrade-package fastapi

# upgrade across a major boundary by editing
# pyproject.toml then re-locking
sed -i 's/pydantic>=2.8/pydantic>=3/' pyproject.toml
uv lock
```

### What changes on disk

- `pyproject.toml` — adds the dependency line in the right table
- `uv.lock` — re-resolves; the diff is your audit trail
- `.venv` — installed/removed via hardlinks from cache

> **Tip** — commit `pyproject.toml` and `uv.lock` together. Reviewers can see both intent and effect in one diff.

---

## Slide 10 — The Lockfile — uv.lock

`uv.lock` is a **universal, cross-platform, multi-Python lockfile**. One file pins your project for Linux x86, Linux arm64, macOS arm64, Windows, and every supported Python version simultaneously.

### Excerpt

```toml
version = 1
requires-python = ">=3.11"

[[package]]
name = "fastapi"
version = "0.115.4"
source = { registry = "https://pypi.org/simple" }
dependencies = [
    { name = "pydantic" },
    { name = "starlette" },
]
sdist = { url = "...", hash = "sha256:abc..." }
wheels = [
    { url = "...py3-none-any.whl", hash = "sha256:def..." },
]

[[package]]
name = "pydantic"
version = "2.9.2"
```

### Why universal matters

- Mac developer machine, Linux containers, Windows build agent — same lockfile
- Resolves once across all platforms; per-platform conditional wheels recorded in the same file
- Compare to Poetry: cross-platform but not multi-Python; pip-tools: per-platform requirements files

### Reproducibility flags

```bash
# refuse to update the lock — fail if it's out of date
uv sync --frozen
uv sync --locked    # alias

# ignore lock and re-resolve from pyproject
uv sync --no-lock
```

### In CI

**Always** run `uv sync --frozen`. If the lockfile is stale the build fails immediately — the way it should.

---

## Slide 11 — uv sync — Reconcile the Environment

### What sync does

1. Ensure `uv.lock` matches `pyproject.toml` (re-lock if not)
2. Ensure `.venv` exists with the right Python
3. Install / upgrade / remove packages so `.venv` matches the lock *exactly*
4. Remove anything in the venv that is not in the lock

**It is idempotent.** Running it twice is essentially free.

### Common flag combinations

```bash
uv sync                       # default groups
uv sync --all-groups          # everything dev + extras
uv sync --no-dev              # production install
uv sync --extra postgres      # one extra
uv sync --group docs          # one dev group
uv sync --frozen              # CI / reproducible
uv sync --reinstall           # nuke and rebuild
uv sync --reinstall-package torch
```

### Performance: the second sync is free

```text
$ time uv sync          # cold
real    0m1.245s
$ time uv sync          # warm
real    0m0.034s        # nothing to do
```

Because uv hashes the lock + venv state, a clean sync becomes a 30 ms no-op. CI pipelines can `uv sync` at the start of every job without paying for it.

### Production install

```dockerfile
ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy
RUN uv sync --frozen --no-dev
```

- `--no-dev` drops dev groups
- `UV_COMPILE_BYTECODE` precompiles `.pyc`
- `UV_LINK_MODE=copy` avoids hardlink issues across volumes

---

## Slide 12 — uv run — Execute in the Project Env

`uv run` auto-syncs the environment, then exec's the command inside it. **No activation, no `poetry shell`**.

### Everyday usage

```bash
uv run python main.py
uv run pytest -q
uv run ruff check .
uv run mypy src/
uv run ipython
uv run python -c 'import torch; print(torch.__version__)'

# with extra runtime deps not in the project
uv run --with rich --with httpx python -c \
  'from rich import print; print({"hi": 1})'

# pin a Python different from .python-version
uv run --python 3.13 pytest
```

### Console scripts

Define entry points in `pyproject.toml`, then `uv run` calls them by name.

```toml
[project.scripts]
serve = "myapp.cli:main"
```

```bash
uv run serve
```

### Behaviour cheat-sheet

- Re-syncs the venv unless `--no-sync`
- Sets `VIRTUAL_ENV`, `PATH`, `PYTHONPATH` for the child process
- Refuses to run if the lock is out-of-date and `--frozen` is set
- Inherits stdin/stdout — works for REPLs, debuggers, dashboards

> **Pitfall** — `uv run python` is not the same as a previously-activated Python. If you've manually `source`'d the venv, drop the `uv run` prefix to avoid double activation.

---

## Slide 13 — Managing Python Versions

uv ships its own Python distribution manager — replacing pyenv, deadsnakes, and the official installers.

### Install & list

```bash
uv python list                # installed + downloadable
uv python list --only-installed
uv python install 3.12.7
uv python install 3.11 3.12 3.13
uv python install 'cpython-3.12.7-linux-x86_64-gnu'
uv python install pypy@3.10

# uninstall
uv python uninstall 3.11

# show where one lives
uv python find 3.12
```

### Pinning

```bash
# write/update .python-version
uv python pin 3.12

# uses the project's .python-version
uv venv

# override per command
uv run --python 3.13 pytest
```

### How is this so fast?

- uv pulls **python-build-standalone** — relocatable, statically-linked CPython tarballs (~25 MB)
- No compilation — just download and untar
- Cached in `~/.local/share/uv/python/` and shared across every project
- Works on locked-down corp laptops where you can't `apt install python3.12`

### System Python vs uv Python

- uv prefers its managed builds for project venvs
- `--system` opts back to the OS interpreter (rare; mostly Docker)
- `UV_PYTHON_PREFERENCE=only-system` if your security policy forbids downloading interpreters

---

## Slide 14 — PEP 723 — Inline Script Dependencies

Single-file scripts can declare their dependencies *inside the script*. uv reads the metadata, builds an ephemeral venv, runs the script.

### A real, runnable example

```python
#!/usr/bin/env -S uv run --script
# /// script
# requires-python = ">=3.12"
# dependencies = [
#   "httpx>=0.27",
#   "rich>=13",
# ]
# ///
import httpx
from rich import print

r = httpx.get("https://api.github.com/zen")
print(f"[bold green]GitHub says:[/] {r.text}")
```

```bash
$ chmod +x zen.py
$ ./zen.py
GitHub says: Practicality beats purity.
```

### Why this is huge

- No `pyproject.toml`, no venv, no `requirements.txt`
- One-file scripts that *actually run* on a colleague's machine
- Replaces a lot of "I made you a notebook"
- Perfect for ops scripts, CI helpers, code-review demos

### Tooling commands

```bash
# add a dep to a script's PEP 723 block
uv add --script zen.py "click>=8"

# pin Python
uv add --script zen.py --python 3.13

# lock script deps separately (optional)
uv lock --script zen.py
```

### Standard, not Astral-only

PEP 723 is an accepted Python standard. Hatch, pdm, and pipx implement it too — your script stays portable.

---

## Slide 15 — uvx & uv tool — Replacing pipx

### One-shot CLI execution

```bash
# run a CLI without installing it
uvx ruff check .
uvx black .
uvx pre-commit run --all-files
uvx httpie GET https://example.com

# specific version
uvx ruff@0.6.9 check .

# from git
uvx --from 'git+https://github.com/foo/bar' baz

# pass extras / extra deps
uvx --with pandas streamlit run app.py
```

### Persistent install (replaces pipx install)

```bash
uv tool install ruff
uv tool install ruff@0.6.9
uv tool install pre-commit --with pre-commit-hooks
uv tool list
uv tool upgrade --all
uv tool uninstall ruff
```

### Where do tools live?

- Each tool gets an isolated venv under `~/.local/share/uv/tools/`
- Console scripts get symlinked into `~/.local/bin` (configurable via `UV_TOOL_BIN_DIR`)
- Add that dir to `PATH` with: `uv tool update-shell`

### `uvx` vs `uv tool install`

| | uvx | uv tool install |
|---|---|---|
| Install location | tmp / cache | persistent |
| On PATH | no | yes |
| Best for | occasional use | daily drivers |

> **Tip** — use `uvx` in CI pre-commit jobs. It avoids polluting the build venv and is faster than `pip install --no-deps pre-commit`.

---

## Slide 16 — Workspaces — Cargo for Python

A **workspace** is a multi-package repo with one shared lockfile, one shared venv, and one resolution pass. Mirrors Cargo / pnpm workspaces.

### Layout

```text
repo/
├── pyproject.toml           # workspace root
├── uv.lock                  # ONE lockfile
├── apps/
│   ├── api/
│   │   └── pyproject.toml
│   └── worker/
│       └── pyproject.toml
└── libs/
    ├── core/
    │   └── pyproject.toml
    └── adapters/
        └── pyproject.toml
```

### Root `pyproject.toml`

```toml
[tool.uv.workspace]
members = ["apps/*", "libs/*"]
exclude = ["apps/legacy"]

[tool.uv.sources]
core      = { workspace = true }
adapters  = { workspace = true }
```

### Cross-package deps

```toml
# apps/api/pyproject.toml
[project]
name = "api"
dependencies = [
  "core",        # path-resolved via workspace
  "adapters",
  "fastapi>=0.115",
]

[tool.uv.sources]
core     = { workspace = true }
adapters = { workspace = true }
```

### Operations

```bash
# sync everything
uv sync

# scope to a member
uv sync --package api
uv run --package api uvicorn api.main:app

# add a dep to one member
uv add --package worker celery
```

Editable cross-package deps without manual `pip install -e` juggling. Refactor across apps and libs with one resolve.

---

## Slide 17 — Dependency Groups & Optional Extras

### Dependency groups (PEP 735)

Internal-only groups for dev tooling, docs, profiling — never installed by your users.

```toml
[dependency-groups]
dev      = ["pytest>=8", "ruff>=0.6"]
docs     = ["mkdocs>=1.6", "mkdocs-material"]
profile  = ["py-spy", "memray"]
typing   = ["mypy>=1.10", "types-requests"]

# include another group inside one
test     = [{ include-group = "dev" }, "pytest-asyncio"]
```

```bash
uv sync --group dev
uv sync --group dev --group typing
uv sync --all-groups
uv run --group test pytest
```

### Optional extras (legacy PEP 621)

Public flags your users opt into when installing your package.

```toml
[project.optional-dependencies]
postgres = ["asyncpg>=0.29"]
redis    = ["redis>=5"]
all      = ["my-app[postgres,redis]"]
```

```bash
# consumer
pip install my-app[postgres]
uv pip install 'my-app[postgres,redis]'
```

### Groups vs extras — when to use which

| | Groups | Extras |
|---|---|---|
| Visible to users | no | yes |
| Use case | dev tooling | opt-in features |
| Standard | PEP 735 | PEP 621 |
| Cmd | `uv sync --group X` | `uv sync --extra X` |

---

## Slide 18 — Build & Publish

### Build wheels & sdists

```bash
# build into ./dist/
uv build

# only one format
uv build --wheel
uv build --sdist

# specify a workspace member
uv build --package my-lib

# inspect
unzip -l dist/my_lib-0.1.0-py3-none-any.whl
```

Replaces `python -m build`. Reads the `build-system` table from `pyproject.toml` — Hatchling, setuptools, maturin (Rust extensions), all work.

### Publish to PyPI

```bash
uv publish                            # uses ~/.pypirc
uv publish --token "$PYPI_TOKEN"
uv publish --publish-url https://test.pypi.org/legacy/
```

Replaces `twine`. Supports trusted publishing (OIDC) from GitHub Actions — no token to manage.

### GitHub Actions: trusted publish

```yaml
name: Publish
on:
  release:
    types: [published]
permissions:
  id-token: write   # OIDC for PyPI
jobs:
  pypi:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v3
      - run: uv build
      - run: uv publish --trusted-publishing automatic
```

### Bumping a version

```bash
uv version 1.2.0
uv version --bump patch
uv version --bump minor
uv version --bump major
```

Edits `pyproject.toml` & refreshes the lock — commit the diff and tag.

---

## Slide 19 — Performance Internals

### The global cache

- One content-addressed store at `~/.cache/uv/` (Linux/macOS) or `%LOCALAPPDATA%\uv\cache`
- Wheels, sdists, source builds, metadata — keyed by hash, deduplicated across projects
- Building NumPy from source once means every other venv on the machine gets it for free
- Inspect: `uv cache dir`, `uv cache prune`, `uv cache clean`

### Hardlinks & reflinks

- Default `UV_LINK_MODE=hardlink` — venv files are filesystem links into the cache
- On btrfs/APFS: `--link-mode=reflink` (copy-on-write)
- Forced full copies: `--link-mode=copy` (Docker volumes, NFS)
- Symlinks for dev: `--link-mode=symlink`

### PubGrub resolver

- SAT-style algorithm with conflict-driven clause learning — same family as Dart pub, npm, Cargo
- Reports human-readable conflicts: "*fastapi 0.115 requires pydantic ≥ 2, but my-lib pins pydantic ≤ 1*"
- Pre-fetches metadata over HTTP/2 in parallel from PyPI's JSON API

### Tunables (env vars)

```bash
UV_CACHE_DIR=/big/disk/uv-cache
UV_LINK_MODE=copy
UV_HTTP_TIMEOUT=120
UV_CONCURRENT_DOWNLOADS=20
UV_NO_CACHE=1
UV_OFFLINE=1
UV_INDEX_URL=https://internal/simple
UV_NATIVE_TLS=1
```

---

## Slide 20 — uv vs pip / Poetry / pdm / Hatch

| Capability | pip | Poetry | pdm | Hatch | uv |
|---|---|---|---|---|---|
| Install / uninstall | ✅ | ✅ | ✅ | ✅ | ✅ |
| Lockfile | — | ✅ | ✅ | — | ✅ |
| Universal lock | — | partial | ✅ | — | ✅ |
| Workspaces | — | — | partial | — | ✅ |
| Manage Python | — | — | partial | ✅ | ✅ |
| Build / publish | — | ✅ | ✅ | ✅ | ✅ |
| PEP 723 scripts | — | — | partial | ✅ | ✅ |
| Run CLIs (pipx) | — | — | — | — | ✅ |
| Single binary | — | — | — | — | ✅ |
| Resolver speed | slow | slow | medium | medium | very fast |

### Verdicts

- **pip** — still the floor, fine for one-off venvs
- **Poetry** — mature, opinionated, slow resolver, no Python management
- **pdm** — closest spiritual ancestor of uv; smaller ecosystem
- **Hatch** — strong on builds + matrices, weaker on locking
- **uv** — covers all of the above, faster, single binary

### When NOT to use uv

- Conda-only stacks (geosciences, some ML) — use pixi or stick with conda
- Legacy projects without `pyproject.toml` — use `uv pip` only
- Air-gapped envs that can't pre-cache wheels — needs care, but doable with `UV_OFFLINE=1`

---

## Slide 21 — Migrating from pip / requirements.txt

### Stage 1 — drop-in (no project changes)

```bash
# nothing to change in the repo, just:
alias pip='uv pip'

uv pip install -r requirements.txt
uv pip compile requirements.in -o requirements.txt
uv pip sync requirements.txt
```

Useful for proving uv works in your CI before any rewrite.

### Stage 2 — add a pyproject.toml

```bash
# interactive, infers from cwd
uv init --name my-app .

# import the existing requirements
xargs -a requirements.txt uv add
```

### Stage 3 — produce uv.lock and delete the old file

```bash
uv lock
git add pyproject.toml uv.lock
git rm requirements.txt requirements.in

# CI now does
uv sync --frozen --no-dev
```

### Keep `requirements.txt` for downstream consumers

```bash
# export from the lock for tools that
# can't read uv.lock yet
uv export --format requirements-txt \
  --no-dev --no-hashes \
  -o requirements.txt
```

Useful for AWS Lambda, legacy Docker base images, IDE remote interpreters, vendor security scanners.

> **Don't** keep both `requirements.txt` and `uv.lock` as sources of truth. Pick one. Export *from* the lock; never edit a generated requirements file by hand.

---

## Slide 22 — Migrating from Poetry

### What changes

| Poetry | uv |
|---|---|
| `poetry install` | `uv sync` |
| `poetry add X` | `uv add X` |
| `poetry remove X` | `uv remove X` |
| `poetry lock` | `uv lock` |
| `poetry run X` | `uv run X` |
| `poetry shell` | `source .venv/bin/activate` |
| `poetry build / publish` | `uv build / uv publish` |
| `[tool.poetry]` | `[project]` + `[tool.uv]` |
| `poetry.lock` | `uv.lock` |

### The `[tool.poetry]` rewrite

```toml
# before
[tool.poetry]
name = "svc"
version = "1.2.3"
authors = ["You <you@x.com>"]
[tool.poetry.dependencies]
python   = "^3.12"
fastapi  = "^0.115"
[tool.poetry.group.dev.dependencies]
pytest = "^8"

# after
[project]
name = "svc"
version = "1.2.3"
requires-python = ">=3.12,<3.14"
authors = [{name = "You", email = "you@x.com"}]
dependencies = ["fastapi>=0.115,<1"]

[dependency-groups]
dev = ["pytest>=8"]
```

### Caret → SemVer specifiers

- Poetry's `^1.2.3` → uv's `>=1.2.3,<2`
- Poetry's `~1.2.3` → uv's `~=1.2.3` (compatible release)
- `poetry-plugin-export` → `uv export --format requirements-txt`

---

## Slide 23 — Configuration — pyproject.toml & uv.toml

### [tool.uv] section

```toml
[tool.uv]
# install the project itself in editable mode
package = true

# always sync these groups by default
default-groups = ["dev"]

# constrain Python versions for the lock
python-preference = "managed"
required-version  = ">=0.5"

# index configuration
index-strategy = "unsafe-best-match"
prerelease     = "if-necessary"

# resolve only certain platforms
[tool.uv.environments]
linux-x86 = "platform_system == 'Linux' and platform_machine == 'x86_64'"

# private indexes
[[tool.uv.index]]
name     = "internal"
url      = "https://pkg.acme.com/simple"
priority = "supplemental"
explicit = true

[tool.uv.sources]
mylib = { index = "internal" }
```

### User-level `uv.toml`

Same keys, no `[tool.uv]` prefix. Lives at `~/.config/uv/uv.toml`:

```toml
cache-dir   = "/big/disk/uv-cache"
link-mode   = "copy"
python-preference = "only-managed"

[[index]]
name = "company"
url  = "https://pkg.acme.com/simple"
```

### Resolution order

1. Command-line flags
2. Environment variables (`UV_*`)
3. Project `pyproject.toml [tool.uv]`
4. User `~/.config/uv/uv.toml`
5. Built-in defaults

### .env / shell rc snippets

```bash
export UV_LINK_MODE=copy
export UV_PYTHON_PREFERENCE=only-managed
export UV_TOOL_BIN_DIR="$HOME/.local/bin"
```

---

## Slide 24 — CI Patterns — GitHub Actions

### Minimal CI workflow

```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python: ["3.11", "3.12", "3.13"]
    steps:
      - uses: actions/checkout@v4

      - uses: astral-sh/setup-uv@v3
        with:
          version: "0.5.4"
          enable-cache: true
          cache-dependency-glob: "uv.lock"

      - run: uv python install ${{ matrix.python }}
      - run: uv sync --frozen --all-groups
      - run: uv run ruff check .
      - run: uv run mypy src/
      - run: uv run pytest -q --cov
```

### Why `setup-uv@v3`

- Caches `~/.cache/uv` keyed on `uv.lock`
- Pins the uv version in one place
- Adds `uv` & `uvx` to `PATH`
- Restores between matrix legs automatically

### Speed-ups

- `UV_COMPILE_BYTECODE=1` — pre-compile `.pyc` at install time, faster cold start in tests
- `uv sync --frozen` instead of `uv lock && uv sync`
- Re-use the same job's venv between steps — no need to rebuild
- Skip `actions/setup-python` entirely; `uv python install` is faster

### Lock-drift gate

```yaml
- run: uv lock --check
  # fails if pyproject.toml changed
  # without re-locking
```

---

## Slide 25 — Cheat Sheet — Daily Commands

| Task | Command |
|---|---|
| Start a project | `uv init my-app` |
| Add a dep | `uv add httpx` |
| Add dev dep | `uv add --group dev pytest` |
| Remove dep | `uv remove httpx` |
| Sync env | `uv sync` |
| Run a script | `uv run python main.py` |
| Run tests | `uv run pytest` |
| Run a one-off CLI | `uvx ruff check .` |
| Install a CLI globally | `uv tool install pre-commit` |
| Pin a Python | `uv python pin 3.12` |
| Re-lock w/ upgrade | `uv lock --upgrade` |
| Reproducible install | `uv sync --frozen` |
| Production install | `uv sync --frozen --no-dev` |
| Build wheels | `uv build` |
| Publish to PyPI | `uv publish` |
| Bump version | `uv version --bump patch` |
| Export to req.txt | `uv export -o req.txt` |
| Inspect dep tree | `uv tree` |
| Why is X here? | `uv tree --invert --package X` |
| Outdated deps | `uv tree --outdated` |
| Cache dir | `uv cache dir` |
| Clean cache | `uv cache prune` |

---

## Slide 26 — Gotchas & Troubleshooting

### Hardlink errors across volumes

Docker bind mounts and NFS often refuse hardlinks: `error: Failed to create hard link`.

```bash
export UV_LINK_MODE=copy
```

### "requires-python" mismatch in CI

Lock was made for `>=3.12` but CI is on 3.11. Either bump CI Python (preferred) or relax `requires-python`.

### Private index 401 in CI

```toml
[[tool.uv.index]]
name = "internal"
url  = "https://pkg.acme.com/simple"
```

```bash
# in CI
UV_INDEX_INTERNAL_USERNAME=ci
UV_INDEX_INTERNAL_PASSWORD=$TOKEN
```

uv reads `UV_INDEX_<NAME>_USERNAME / _PASSWORD`.

### PyTorch & CUDA wheels

```toml
[tool.uv.sources]
torch = [
  { index = "pytorch-cu124", marker = "platform_system != 'Darwin'" },
  { index = "pypi",          marker = "platform_system == 'Darwin'" },
]

[[tool.uv.index]]
name     = "pytorch-cu124"
url      = "https://download.pytorch.org/whl/cu124"
explicit = true
```

### "Cache is huge"

```bash
uv cache dir
uv cache prune          # safe — keeps recent
uv cache clean          # nuke everything
```

### Editable install of the project itself

If you want `import myapp` to work after `uv sync`, add:

```toml
[tool.uv]
package = true
```

---

## Slide 27 — Summary & Next Steps

### What we covered

- uv as the all-in-one Python toolchain
- `uv pip` as the drop-in path; `uv add` as the project path
- Universal multi-platform `uv.lock`
- `uv sync` & `uv run` for everyday work
- Managing Pythons, PEP 723 scripts, and `uvx` tools
- Workspaces, dep groups, build & publish
- Performance internals — cache, hardlinks, PubGrub
- Migration from pip-tools and Poetry
- CI patterns & gotchas

### Recommended next steps

1. Install uv and run `uv pip` against an existing project
2. Convert one repo with `uv init .` + `uv add`
3. Replace `actions/setup-python` with `astral-sh/setup-uv`
4. Pick one ops script, give it a PEP 723 header, ship it
5. Move dev tools (`ruff`, `pre-commit`) under `uv tool install`
6. For monorepos: try `[tool.uv.workspace]`

### Companion deck

For Docker, multi-stage builds, GPU/PyTorch recipes, monorepo workflows, and migration playbooks see **"uv in Practice"**.

### Further reading

- docs.astral.sh/uv
- github.com/astral-sh/uv
- python-build-standalone — github.com/indygreg/python-build-standalone
- PEP 723 — peps.python.org/pep-0723
- PEP 735 — peps.python.org/pep-0735

### One-line takeaway

If you're still typing `python -m venv`, `pip install -r`, or `poetry install` — try `uv sync` once. You won't go back.
