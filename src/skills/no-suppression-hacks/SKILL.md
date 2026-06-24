---
name: no-suppression-hacks
description: The binding "fix the cause, never silence the symptom" rule that every code-writing agent applies whenever it writes, modifies, or proposes code. Load before producing any code edit. Forbids suppression comments and gate-bypass hacks used to make a linter, type checker, security scanner, formatter, or test gate go quiet without fixing the underlying defect — # noqa, bare # type: ignore, # pyright: ignore, # pylint: disable, # nosec, # pragma: no cover, # fmt: off/skip, eslint-disable, and blanket ignore entries in tool config. Also forbids the related shortcuts of swallowing exceptions, deleting or skipping tests, weakening assertions or types, and bypassing safety gates (--no-verify, --force, disabling hooks) to reach a green state. Defines the single narrow exception path: a scoped suppression with an error code, a one-line justification, and an "R. Penno" tag, used only when the tool is provably wrong.
user-invocable: false
context: fork
---

# Fix the Cause, Never Silence the Symptom

When a linter, type checker, security scanner, formatter, or test gate flags your
code, the **only** acceptable response is to diagnose and fix the root cause. A
warning is a signal that something is wrong — silencing the signal does not make
the code correct, it makes the defect invisible. Suppressing a tool to reach a
green state is a defect in its own right, and a change that does it is refused.

This rule is binding for every code you write, modify, or propose. It is not a
stylistic preference.

## Forbidden by default — suppression comments

Never insert a suppression comment to make a tool stop complaining:

- `# noqa` / `# noqa: <code>` (ruff, flake8)
- bare `# type: ignore` (mypy)
- `# pyright: ignore` / `# pyright: ignore[reportX]` used as a shrug
- `# pylint: disable=...`
- `# nosec` (bandit)
- `# pragma: no cover` (coverage)
- `# fmt: off` / `# fmt: skip` (black) used to dodge formatting
- `// eslint-disable` / `// eslint-disable-next-line` and equivalents in other languages
- `# type: ignore`-style blanket markers at module top

## Forbidden by default — config-level silencing

The same prohibition applies one level up, in tool configuration. Do not reach a
green state by:

- adding blanket ignores to `[tool.ruff]` / `[tool.ruff.lint] ignore`, `[tool.mypy]` `disable_error_code`, `per-file-ignores`, or `exclude`
- expanding `[tool.coverage.run] omit` or lowering `--cov-fail-under`
- relaxing `pyrightconfig.json` / `mypy.ini` severities to hide a real error
- loosening a dependency version constraint purely to make a resolver or checker stop erroring

## Forbidden by default — gate-bypass shortcuts

Reaching "green" by removing the check is the same sin as suppressing it:

- swallowing exceptions (`except: pass`, broad catches, or downgrading a raise to a log) to make a failure disappear — catch the **specific** exception, log it, and re-raise
- deleting, commenting out, `@pytest.mark.skip`-ing, or `xfail`-ing a failing test to get a passing run
- weakening an assertion, loosening a type to `Any`, or broadening a return annotation just to satisfy a checker
- bypassing safety gates: `--no-verify`, `git push --force`, `git commit --amend` on pushed work, disabling pre-commit hooks, or passing `--ignore` flags to skip checks

When a tool fails, the cause is one of: a real defect in the code (fix the code),
a stale or misconfigured tool (fix the configuration honestly, for everyone), or
a genuine tool bug (the narrow exception below). It is never "add a suppression
and move on."

## The single narrow exception

A scoped suppression is allowed **only** when the tool is provably wrong and the
underlying code is genuinely correct — for example a third-party stub bug or a
checker false positive you have verified. When you use it, all three of the
following are mandatory:

1. **Scope it to the specific error code** — `# type: ignore[attr-defined]`, not
   bare `# type: ignore`; `# noqa: E501`, not bare `# noqa`.
2. **Justify it in a one-line comment** stating *why* the tool is wrong (cite the
   upstream issue, the false-positive reason, or the API constraint).
3. **Tag it `R. Penno`** so the suppression is auditable and can be removed when
   the upstream bug is fixed.

A suppression missing any of these three is forbidden. When you finish, report
the count of any suppressions you added and the justification for each. If the
count went up, say so explicitly and explain why.
