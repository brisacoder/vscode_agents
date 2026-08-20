# Changelog

All notable changes to this repository's agents and skills are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

History prior to this file's introduction is available in `git log`; this changelog
starts from the point it was added rather than reconstructing every prior change.

## [Unreleased]

### Added

- `PR Stack Planner` agent (`pr-stack-planner.agent.md`), replacing `PR Discipline Expert`.
  Plan mode is now the primary role: the stack is planned up front, before any code is
  written, instead of being shaped too late at submit time. Enforce/Review/Fix modes
  remain as the downstream safety net for the same six PR-shape rules.
- `Code Review Generalist` agent (`code-review-generalist.agent.md`) — a checklist-free,
  "fresh eyes" reviewer with no domain lane and no "out of scope, delegate" rule. Carries
  the lowest dedup precedence of any reviewer, so it only ever contributes net-new findings
  that fell between specialists' checklists.
- `stack-shepherding` skill — the mandatory, non-negotiable contract for what happens to a
  pull-request stack *after* it is submitted: auto-merge on every PR, every review comment
  addressed and its thread resolved, every PR carrying a Jira ticket key, and
  submitted-and-green treated as the middle of the job rather than the end. Loaded by
  `PR Stack Planner`, `PR Watch Agent`, and `PR Review Resolver`.
- `argument-hint` frontmatter added to every agent that was missing it (10 agents), so the
  chat input placeholder is populated consistently across the whole agent roster.
- "Skills" section in `README.md` documenting what skills are, where to install them, and
  which agents depend on which skill.

### Changed

- Migrated the stacked-PR workflow from the third-party Graphite CLI (`gt`) to GitHub's
  native stacked pull request feature, driven locally via the `gh stack` CLI extension.
  The `graphite-stacking` skill was replaced by `github-stacking`; every `gt <command>`
  reference across `pr-stack-planner`, `pr-watch-agent`, `pr-review-resolver`,
  `code-authoring-executor`, and `code-review-executor` was rewritten to its `gh stack`
  equivalent, including the semantic difference that `gh stack` does not auto-cascade a
  commit on a lower branch the way Graphite's `gt modify` did.
- Migrated the model roster referenced in agent handoffs and dispatch tables from
  `Claude Opus 4.7` / `Gemini 3.1 Pro Preview` to `Claude Sonnet 5` / `Gemini 3.5 Flash`.
- Unified the Finding ID Prefix contract across `code-reviewer-agent`, `code-review-executor`,
  and the `consolidated-review-report` skill. All three now use the same dashed,
  multi-letter prefix format (e.g. `PY-`, `DOC-`, `PR-`, `GEN-`) — previously
  `code-reviewer-agent` used an undashed, single/double-letter format that both
  disagreed on structure and, for several prefixes, mapped to a different specialist
  entirely (e.g. `DOC` meant README Expert in one table and `DOC-` meant Docstring
  Expert in the other).
- Deduplicated the `tools:` frontmatter array in 14 agent files where `'github/*'` (13
  files) or `agent` (1 file) was listed twice.
- Narrowed `agents: ["*"]` on 12 leaf specialist agents (AWS, CI/CD, Docker, FastAPI, GCP,
  Logic and Correctness, Observability, PyArrow, Pydantic, Python, PyTorch, Scikit-learn
  Experts) that never dispatch a specifically-named different agent — they only ever
  self-spawn anonymous verifier/hunter subagents for their own saturation review loop.
  Bringing them in line with the dozen agents that already omitted the field.
- Trimmed the `saturation-review-loop` skill's frontmatter `description` from 1,551 to
  942 characters to respect the Agent Skills spec's 1,024-character limit. The examples
  that were removed from the description remain documented in full in the skill body.

### Removed

- `pr-discipline-expert.agent.md`, superseded by `pr-stack-planner.agent.md`.
