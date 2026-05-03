# 🐍 Introduction to uv

An interactive Reveal.js presentation covering **uv** — the extremely fast, all-in-one Python package and project manager from Astral. Replaces pip, pip-tools, pipx, poetry, pyenv, virtualenv, and twine with a single Rust binary.

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/Introduction_to_uv/)

## 📄 [Markdown Version](presentation.md)

## 🧪 [Companion deck — uv in Practice](https://brendanjameslynskey.github.io/uv_in_Practice/)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Title | One-tool pitch and the resolve→lock→sync→run flow |
| 02 | Topics | Map of foundations, project workflow, power features |
| 03 | What Is uv? | Astral, Rust, and the list of tools uv replaces |
| 04 | Why uv? | Speed numbers and the engineering behind them |
| 05 | Installation | Standalone installer, brew/winget, Docker, self-update |
| 06 | uv pip | Drop-in pip replacement and `pip-tools` workflow |
| 07 | uv venv | Virtual environments in 50 ms |
| 08 | uv init | Project scaffolds — app, lib, package |
| 09 | pyproject.toml | Source of truth — deps, groups, extras, specifiers |
| 10 | uv add / remove | Everyday verbs and the resulting on-disk changes |
| 11 | uv.lock | Universal multi-platform multi-Python lockfile |
| 12 | uv sync | Idempotent reconcile and production install patterns |
| 13 | uv run | Execute in-project without activating a venv |
| 14 | Python versions | `uv python install/pin` and python-build-standalone |
| 15 | PEP 723 scripts | Single-file scripts with inline dependency metadata |
| 16 | uvx & uv tool | One-shot CLIs and persistent installs (replaces pipx) |
| 17 | Workspaces | Cargo-style monorepos with one lockfile |
| 18 | Dep groups & extras | PEP 735 vs PEP 621 — when to use which |
| 19 | Build & publish | `uv build`, `uv publish`, OIDC trusted publishing |
| 20 | Performance internals | Cache, hardlinks, PubGrub, env-var tunables |
| 21 | uv vs pip/Poetry/pdm/Hatch | Capability matrix and verdicts |
| 22 | Migrating from pip | Three-stage path from `requirements.txt` to `uv.lock` |
| 23 | Migrating from Poetry | Command map and `[tool.poetry]` rewrite |
| 24 | Configuration | `[tool.uv]`, user-level `uv.toml`, env vars |
| 25 | CI Patterns | GitHub Actions with `astral-sh/setup-uv@v3` |
| 26 | Cheat Sheet | Daily commands, two-column reference |
| 27 | Gotchas | Hardlinks, private indexes, PyTorch wheels, cache size |
| 28 | Summary | Takeaways, next steps, further reading |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## References

uv documentation — docs.astral.sh/uv · uv source — github.com/astral-sh/uv · python-build-standalone — github.com/indygreg/python-build-standalone · PEP 723 (inline scripts) — peps.python.org/pep-0723 · PEP 735 (dependency groups) — peps.python.org/pep-0735 · PyPA packaging guide — packaging.python.org

## License

Educational use. Code examples provided as-is.
