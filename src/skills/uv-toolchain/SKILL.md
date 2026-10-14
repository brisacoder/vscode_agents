---
name: uv-toolchain
description: Canonical uv command set for Python tooling in this workspace. Load whenever an agent needs to run tests, format code, lint, type-check, install dependencies, or execute Python scripts. The workspace standard forbids global pip install and bare python invocations; everything goes through uv so the project's locked environment is used. Covers uv run pytest, uv run black, uv run isort, uv run ruff check, uv run mypy / pyright, uv add, uv pip install -e, uv sync, and uv run python script.py. Apply for any Write Mode, Optimize Mode, Quality Gate, or Reconciliation step that needs to verify tests pass, formatting is clean, lint is green, type checks pass, or a script runs in the correct environment.
user-invocable: false
context: inline
---

# uv Toolchain — Canonical Commands

This workspace is uv-managed. Every Python tool runs through `uv run` (which activates the project's locked virtual environment automatically) or `uv` itself. Do not invoke `python`, `pip`, `pytest`, `black`, `isort`, `ruff`, `mypy`, or `pyright` directly — those resolve against the wrong interpreter and miss locked dependencies.

## Mandatory implementation-first gate

Load `implementation-first` before coding or test execution. **DO NOT run/create/change tests during implementation; finish the agreed code and hand it to the user first.**
Commands below are syntax references, not authorization. Baseline suites, doctests, coverage, collection, and per-edit test cycles require the approved test phase.

## Test execution

- **Full test suite (concise)**: `uv run pytest --tb=line -q`
- **Verbose run**: `uv run pytest -v`
- **Scoped to a file or directory**: `uv run pytest -v <path>`
- **Doctests**: `uv run pytest --doctest-modules <path>`
- **Coverage (per package, with the workspace 75% gate)**: `uv run pytest --cov=<package> --cov-report=term-missing --cov-fail-under=75 -q`
- **Surface deprecations as errors**: `uv run pytest -W error::DeprecationWarning -W error::PendingDeprecationWarning`

## Formatters (mandatory before every commit and PR-open)

- `uv run black <files>` — apply
- `uv run black --check <files>` — verify, no edits
- `uv run isort <files>` — apply
- `uv run isort --check <files>` — verify, no edits

A file that would fail `black --check` or `isort --check` is not done. This applies to source files, test files, scripts, and ad-hoc utilities equally.

## Linters and type checkers

- `uv run ruff check` — workspace lint
- `uv run ruff check --select F401,F821 <files>` — unused-import / undefined-name only
- `uv run mypy --strict <module>` — strict type check
- `uv run pyright <module>` — alternative type check when configured

## Dependency management

- `uv add <package>` — add a runtime dependency (writes to `pyproject.toml` and `uv.lock` together)
- `uv add --dev <package>` — add a dev dependency
- `uv add -e ./path/to/local-package` — local editable install (workspace standard #13)
- `uv sync` — install / refresh the locked environment
- `uv lock` — refresh the lockfile after a `pyproject.toml` change without installing

`pyproject.toml` and `uv.lock` must move together in the same commit. Never strip a version constraint to silence a resolver error — diagnose it.

## Script execution

- `uv run python <script.py>` — run a Python script
- `uv run python -c "..."` — quick inline Python
- `uv run python -m py_compile <file>` — verify a file parses
- `uv run <console-script>` — any console_scripts entry from `pyproject.toml`

## Phase-aware quality checks

After the complete implementation batch, run necessary static checks on touched files:

1. `uv run black <files>` (apply)
2. `uv run isort <files>` (apply)
3. `uv run ruff check --select F401,F821 <files>` (fix unused imports / undefined names)
4. Hand off `implementation complete — awaiting user validation; automated tests deferred`.

Only after user test-phase authorization: run the named affected selection once, then changed failing selections after corrective batches.
Coverage/full-suite commands run only for explicitly authorized/required scope in that approved phase; respect the project's configured floor and preserve unsuppressed failures.
An observed CI failure is evidence to diagnose, not permission to launch broad local suites or generate tests during implementation. Do not claim deferred checks passed.

## Forbidden invocations

Do not run these in this workspace:

- `pip install ...`, `pip uninstall ...` — bypass the lock
- `python <script.py>`, `python -m pytest` — wrong interpreter
- `pytest`, `black`, `isort`, `ruff`, `mypy`, `pyright` without the `uv run` prefix — wrong environment

If a tool isn't available through `uv run`, add it as a dev dependency (`uv add --dev <tool>`) first.
