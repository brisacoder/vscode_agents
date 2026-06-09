---
name: workspace-standards-preread
description: The mandatory two-step preamble every code-writing and code-reviewing agent runs before touching Python code in this workspace. Step 1 reads the workspace coding standards (.github/copilot-instructions.md, CLAUDE.md if present, and any equivalent) so the agent applies the project's binding rules from the first line. Step 2 reads pyproject.toml requires-python to pin the Python version floor, which gates every version-tagged recommendation ([3.12+], [3.13+], [3.14+]). Load at the start of any Write Mode, Optimize Mode, Rewrite Mode, or Review Mode procedure on a Python target. Failing to read these makes recommendations invalid because they may violate house rules or cite features the locked Python version does not have.
user-invocable: false
context: fork
---

# Workspace Standards Pre-Read

Before writing, modifying, or reviewing any Python code in this workspace, read both of the following. The remainder of the agent's procedure depends on what they say.

## Step 1 — Read the workspace coding standards

Read, in this order, the first one that exists:

1. `.github/copilot-instructions.md` — primary source of binding rules for this workspace
2. `CLAUDE.md` at the repo root — equivalent file used by some projects
3. Any other instruction file the workspace lists in its agent customization config

These files contain the workspace's non-negotiable conventions: package manager (`uv`), import discipline (no import guards, no imports inside functions, no inner functions), formatting (`black`, `isort`), error handling (catch specific exceptions, log AND re-raise, no bare `except:`), logging (`logging`/`rich`, never `print` for diagnostics), serialization (Parquet, never `pickle`), and atomic state mutation. Every recommendation, every fix, every example must honor these rules from the first line written.

If neither file exists, note that in the agent's Reflection Log and proceed with the workspace defaults documented in this skill plus the Zen of Python.

## Step 2 — Read `pyproject.toml` `requires-python`

Open `pyproject.toml` at the repo root and read the `[project]` table's `requires-python` field. The lower bound of that constraint is the **Python version floor**. It gates every version-tagged recommendation.

- `requires-python = ">=3.12"` → 3.12+ features available; 3.13/3.14-tagged recommendations are invalid
- `requires-python = ">=3.13"` → 3.13+ features available
- `requires-python = ">=3.14"` → all current features available (including PEP 750 t-strings, `pathlib.Path.copy()`, `uuid.uuid7()`)

Record the version floor explicitly in any review report header. A `[3.14+]` finding filed against a project pinned to `>=3.12` is a defect — the suggested API does not exist in the locked runtime.

**Do not use `.python-version`** as the version ceiling. That file pins the developer's local interpreter, not the code's minimum compatibility target. Always use `requires-python` from `pyproject.toml`.

## When to skip

This pre-read is mandatory for every Python code path. The only exceptions:

- Pure documentation tasks that don't touch code (README rewrites that don't include code examples)
- Diagram-only tasks (`.drawio` editing, mermaid rendering) with no code changes
- Configuration-only tasks (Dockerfile, GitHub Actions YAML) where Step 1 still applies but Step 2 doesn't

In every other case, both steps run before the agent proceeds.
