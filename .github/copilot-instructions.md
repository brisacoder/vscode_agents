# Coding Standards — MANDATORY

The Zen of Python governs all decisions not explicitly covered below.

---

## Repository Purpose — READ FIRST

This repository is a **storage and distribution library** for reusable Copilot
agents (`src/vscode_agents/`) and skills (`src/skills/`). These artifacts are
authored here, then **copied into other developers' `~/.copilot` directories**
to be used in their own, separate repositories and projects.

Consequences you MUST internalize:

- The agents and skills in this repo are **products shipped to other repos**.
  They are NOT meant to operate on this repo's own code.
- **Do NOT invoke these agents or skills while working inside this repository.**
  Running them here is off-goal and wastes time and tokens — this repo is not
  their target environment.
- When asked to create or modify an agent or skill, treat it as **editing a
  document/specification** (Markdown content), not as a workflow to execute here.
- Their instructions, examples, and assumptions are written for the consuming
  projects, not for this repository.

---

## Non-Negotiable Rules

1.  NO import guards (`try/except` import patterns)
2.  NO imports inside functions or methods
3.  NO inner functions — all functions at module or class level
4.  NO logic outside functions or classes (only constants, logger, `__all__`, type aliases)
5.  NO circular imports — fix by refactoring
6.  NO pickle — use Parquet for data serialization
7.  NO bare `except` clauses
8.  NO `print` for diagnostic output — use `logging`; use `rich` for CLI output
9.  NO emojis in code, commits, or docs
10. Activate virtual environment before any coding or testing
11. Clean up all temporary files

## Package Management

12. Use `uv` exclusively — `uv pip install`, `uv run python`, `uv run <cmd>`

## Project / Import / Style

13. Local packages: `uv add -e ./path/to/package`
14. `snake_case` for all Python files and directories
15. Test files: `test_<module_name>.py`
16. Absolute imports only
17. One `import` / `from` per line; parentheses for multi-name imports
18. Import ordering: stdlib → third-party → local (blank line between groups)
19. Alphabetical within each group
20. Type hints on EVERY parameter and return value
21. PEP 8 compliance
22. Fix ALL linting errors before done
23. Maximum line length: 200 characters
24. Descriptive variable names over comments

## Docstrings

25. Required on: modules, classes, functions, methods
26. Google-style format
27. Must include: description, Args, Returns (if non-None), at least one Example
28. Dunder methods do not require Examples
29. Generic boilerplate docstrings are a defect
30. Update docstrings when changing signatures

## Comments

31. Every comment must add information code cannot convey
32. Tag important comments and TODOs with "R. Penno"
33. AI-generated boilerplate comments are a defect — remove them

## Error Handling & Logging

### Exceptions

34. Catch **specific** exceptions only
35. Every caught exception must be logged AND re-raised. If an error is not
    worth re-raising, do not catch it — log the relevant condition instead
36. Errors must never pass silently

### State Integrity

37. Validate ALL preconditions before ANY state mutation begins. Validation and
    mutation must never be interleaved in the same loop or sequence.
38. Multi-step state changes must be atomic: either all succeed or none are applied.
    Use copy-and-replace or explicit rollback to guarantee this.
39. A function that raises an exception must not leave observable state partially
    modified. If rollback is impractical, document the non-atomic window explicitly.
40. Related data structures that must remain consistent (e.g., a dict and an index)
    must be modified in the same logical transaction.

### Logging

41. Use structured logging with levels: DEBUG, INFO, WARNING, ERROR, CRITICAL
42. Include context: function name, relevant IDs, operation being performed
43. Log before risky operations: "Attempting to connect to database..."
44. Log after critical operations: "Successfully processed 150 records"

### Error Messages

45. Must be actionable: "Failed to connect to database at localhost:5432.
    Check if service is running."
46. Include relevant context: user ID, file path, operation attempted
47. Format: "Operation failed: specific reason. Suggested action."
48. Tag critical errors with `R. Penno`

## Testing

49. Use `pytest` exclusively; integration tests: `@pytest.mark.integration`

## Documentation

50. Every package has a README.md with: purpose, install, usage, architecture
51. Keep CHANGELOG.md up to date
52. Use semantic versioning
53. Document breaking changes with migration instructions
54. API docs generated from docstrings

## Compliance

55. Compliance checklist MUST appear inline in every chat response
56. NEVER save the compliance report to disk
57. Every completed task gets its own checklist; skipping = defect
58. Follow the Zen of Python for all decisions not covered above
