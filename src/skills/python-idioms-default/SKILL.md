---
name: python-idioms-default
description: The Zen of Python tiebreaker and the five-rule idiomatic ranking that all code-writing and code-reviewing agents apply when multiple correct solutions exist. Load whenever writing, reviewing, optimizing, or recommending Python 3.12+ code so that the default choice is always the most explicit, simple, readable, modern, and idiomatic alternative. Covers stdlib-first patterns (pathlib, itertools/functools/contextlib, collections.Counter/deque/defaultdict), modern type syntax (X | None, list[X], type X =, Self, @override, LiteralString), modern OOP and concurrency idioms (Protocol over ABC, @dataclass(slots=True, frozen=True), match over isinstance chains, asyncio.TaskGroup over gather, asyncio.timeout over wait_for), and the deprecated constructs that must be rejected by default (Optional[X], List[X], os.path.* where pathlib fits, datetime.utcnow(), bare except:, for i in range(len(x)), string concatenation in hot loops).
user-invocable: false
context: inline
---

# Default to Idiomatic, Modern Python

When more than one correct solution to an issue exists, your default MUST be the one that best honors the Zen of Python (`import this`): explicit, simple, readable, modern, and idiomatic on the targeted Python version. This is a binding rule, not a stylistic preference.

When ranking alternatives:

1. **Zen of Python is the tiebreaker.** Prefer explicit over implicit, simple over complex, flat over nested, sparse over dense, readability over cleverness. If two solutions are equally correct, the more Pythonic one wins.
2. **Prefer stdlib over third-party** when the stdlib answer is competitive: `pathlib` over `os.path`, `itertools` / `functools` / `contextlib` over manual loops and boilerplate, `collections.Counter` / `deque` / `defaultdict` over hand-rolled dict patterns, `datetime.UTC` over `datetime.utcnow()`.
3. **Prefer modern type syntax** on the targeted Python version: `X | None` over `Optional[X]`, `list[X]` over `List[X]`, `type X =` over `TypeAlias`, `Self`, `@override`, `LiteralString`.
4. **Prefer modern OOP and concurrency idioms**: `Protocol` over `ABC` where structural typing fits, `@dataclass(slots=True, frozen=True)` over plain classes for value objects, `match` over long `isinstance` chains, `asyncio.TaskGroup` over `asyncio.gather`, `asyncio.timeout` over `asyncio.wait_for`.
5. **Reject deprecated and non-idiomatic constructs by default**: never `Optional[X]`, `List[X]`, `os.path.*` where `pathlib` fits, `datetime.utcnow()`, bare `except:`, `for i in range(len(x))`, string concatenation in hot loops where `"".join()` fits.

When you propose, write, review, or recommend a fix and multiple correct options exist, surface the most idiomatic one as the default. If you select a less-Pythonic option, state the explicit reason — measured performance constraint, library API requirement, or project convention — in the same response.

## Hard file-size constraint

**No single `.py` file — source or test — may exceed 300 lines.** This is a CI gate, not a guideline. When writing or reviewing code:

- If a source module approaches 300 lines, split it into focused sub-modules before it crosses the limit.
- If a test file approaches 300 lines, split it into multiple `test_<module>_<aspect>.py` files (e.g., `test_router_happy_path.py`, `test_router_error_cases.py`).
- When reviewing, any file exceeding 300 lines is a **High** severity finding regardless of content quality.
- Shared fixtures that multiple test files need go into `conftest.py` (which is also subject to the 300-line cap — use multiple `conftest.py` at different directory levels if needed).
