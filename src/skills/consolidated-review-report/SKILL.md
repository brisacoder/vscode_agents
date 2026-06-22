---
name: consolidated-review-report
description: Defines the exact structure, sections, ordering, naming conventions, and formatting rules for the consolidated code-review report produced by Code Reviewer V3. Load this skill when assembling, rendering, or rewriting the final consolidated report from specialist findings. Covers report header, Dispatch Summary table, Static pre-analysis section, Cross-model agreement themes, File coverage, per-specialist verbatim inlining, and the Prioritized Summary. Eliminates on-the-fly decisions about section ordering, severity lettering, ID conventions, table columns, and verbatim-boundary markers.
user-invocable: false
context: fork
---

# Consolidated Review Report — Format Specification

This skill defines the exact, reproducible structure of the consolidated code-review report. The orchestrator (Code Reviewer V3) MUST follow this format when rendering the report. No improvisation on section names, ordering, table shapes, or marker conventions is permitted.

## File naming

```
./pr_reviews/code-review-<sanitized-path>-<YYYY-MM-DD>.md
```

Sanitize: replace `/` with `_`, strip leading dots. One file per (reviewed-path, date) pair.

---

## Report skeleton (section ordering is binding)

Render sections in exactly this order. Do not reorder, merge, or skip any section. Sections with no content render their header plus a one-line "None." note.

```markdown
# Code Review: <path reviewed>

**Date**: <YYYY-MM-DD>
**Scope**: <N source files (K src + M tests), ~L LOC>
**Status**: <status-line>
**Last update (UTC)**: <ISO 8601 timestamp>
**Ledger**: `./pr_reviews/.code-review-ledger-<sanitized-path>-<YYYY-MM-DD>.json`

## Static pre-analysis (Areas of Concern shared with specialists)

## Dispatch Summary

## Cross-model agreement themes (best-effort)

## File coverage

## Findings by Specialist

## Orchestrator notes

## Specialist Review Triggers
```

---

## Section 1: Report header

| Field | Content |
|---|---|
| `# Code Review:` | The path that was reviewed (verbatim as given by user) |
| `**Date**` | `YYYY-MM-DD` of the review |
| `**Scope**` | File count breakdown and approximate LOC. Format: `N source files (K src + M tests), ~L LOC` |
| `**Status**` | One of: `Review in progress -- K of N specialists complete (M running, P pending, F failed terminal: <details>)` or `Review complete -- N of N specialists complete (F failed terminal)` |
| `**Last update (UTC)**` | ISO 8601 with seconds. Updated on every ledger write |
| `**Ledger**` | Path to the JSON ledger file, in backticks |

---

## Section 2: Static pre-analysis

Bullet list of deterministic checks run before dispatch. Each bullet names the tool/command, what was checked, and the result. Example format:

```markdown
## Static pre-analysis (Areas of Concern shared with specialists)

- **ruff** (E711, E712, B006, B007, B008, B017, B023, B904): all checks passed
- **module-level executable calls** (AST scan of `src/*.py`): none
- **logging wrapper**: every src module uses `from snt_logging import get_logger`
- **large modules**: `src/package/report.py` (831 LOC) flagged for cohesion review
- **async/sync boundary**: `src/package/run.py` uses `ExitStack` + `asynccontextmanager`
- **No direct LangGraph / FastAPI imports** -- specialists for those did NOT trigger
```

If a check was skipped (tool unavailable), state why. Never omit the section.

---

## Section 3: Dispatch Summary

### Table format (columns are fixed)

```markdown
## Dispatch Summary

`Reported` is the specialist's own count. `Parsed for dedup` is the count of findings whose structured `**ID**: ... **Location**: ...` shape matched the dedup parser. A gap between the columns is expected and not a defect -- the verbatim inlined content below is the authoritative record.

| Specialist | Model | State | Reported | Parsed for dedup | Report path |
|---|---|---|---|---|---|
```

### Column definitions

| Column | Content |
|---|---|
| Specialist | Agent name (e.g. `Python Expert`, `Logic & Correctness Expert`) |
| Model | Model name and vendor (e.g. `Claude Opus 4.7`, `GPT-5.4`, `Gemini 3.1 Pro Preview`) |
| State | One of: `done`, `done (after N retry)`, `running`, `pending`, `failed (terminal after N attempts)`, `not triggered` |
| Reported | Integer count from the specialist's own summary, or `unstated`, or `--` for failed/not-triggered |
| Parsed for dedup | Integer count of findings that matched structured format, or `--`, or `0 (reason)` |
| Report path | Backtick-wrapped path to the specialist's findings file, or `--` |

### Totals line (immediately after the table)

```markdown
**Totals** (as of last ledger write): X reported findings across Y successful specialists; Z parsed into the dedup index. The reported/parsed gap reflects format heterogeneity in [details]. [N] specialist(s) failed terminal ([details]) and [coverage note].
```

---

## Section 4: Cross-model agreement themes

This section is a **best-effort** cross-model agreement view -- explicitly labeled as such. It is NOT the authoritative finding list.

### Table format

```markdown
## Cross-model agreement themes (best-effort)

This section is a best-effort cross-model agreement view extracted from the parsed findings -- it is NOT the authoritative finding list. The authoritative list is the inlined verbatim sections in the next section. Use this table to spot multi-model consensus issues; use the inlined sections to read every finding.

The strongest cross-model consensus themes (3-of-3 or 2-of-3 model agreement) are:

| Theme | Severity | Confidence | Locations | Models in agreement |
|---|---|---|---|---|
```

### Column definitions

| Column | Content |
|---|---|
| Theme | Short description of the recurring pattern (bold) |
| Severity | Highest reported severity across agreeing models (High / Medium / Low) |
| Confidence | `High (3/3)` or `Medium (2/3)` or `Low (1/3)` |
| Locations | Key file:line references (comma-separated) |
| Models in agreement | Which specialist+model combinations agree |

### Footer

```markdown
Confidence key: **High** = 3/3 models agreed; **Medium** = 2/3 models; **Low** = 1/3 model only. Severity reflects the highest reported across the agreeing models.
```

---

## Section 5: File coverage

One line per statement about coverage:

```markdown
## File coverage

All N source files (`src/package/*.py` and `tests/*.py`) are referenced by at least one specialist finding. No unreviewed files.
```

Or, if gaps exist:

```markdown
## File coverage

M of N source files have at least one specialist finding. Unreviewed files: `path/to/file1.py`, `path/to/file2.py`.
```

---

## Section 6: Findings by Specialist

This is the bulk of the report. One subsection per dispatched (specialist, model) row.

### Per-specialist subsection format

```markdown
### <Specialist Name> -- <Model Name>

**State**: done | failed (terminal after N attempts)
**Reported by specialist**: N findings
**Parsed for dedup**: M
**Source file**: `<path to findings file>`

<!-- begin verbatim: <path to findings file> -->

[FULL VERBATIM CONTENT OF THE SPECIALIST'S FINDINGS FILE -- UNCHANGED]

<!-- end verbatim -->
```

### Rules for verbatim inlining

1. Copy the specialist's findings file content **exactly as-is** between the markers
2. Preserve original markdown structure: their headers, tables, prose, severity labels, code blocks
3. Do NOT reformat, summarize, strip, or normalize
4. Do NOT add your own commentary inside the verbatim block
5. The `<!-- begin verbatim: path -->` and `<!-- end verbatim -->` markers are mandatory boundary delimiters
6. Length is not a constraint -- a 100-page specialist report stays 100 pages in the consolidated report
7. For `failed` rows, inline the fallback artifact (raw response + failure reason) under the same markers

### For pending/running specialists

```markdown
### <Specialist Name> -- <Model Name>

**State**: running | pending
<pending>
```

### Ordering of subsections

Order subsections by specialist domain (alphabetical within domain group), then by model within each specialist. Recommended grouping:

1. Python Expert (Claude, GPT, Gemini)
2. Logic & Correctness Expert (Claude, GPT, Gemini)
3. Docstring Expert (Claude, GPT, Gemini)
4. Type Annotation Expert (Claude, GPT, Gemini)
5. README Expert (Claude, GPT, Gemini)
6. Unit Test Expert (Claude, GPT, Gemini)
7. Framework-specific experts in alphabetical order (FastAPI, LangGraph, Pandas, Pydantic, etc.)
8. Observability Expert (Claude, GPT, Gemini)
9. Spec Author (Claude, GPT, Gemini)
10. Architecture Diagram Creator (Claude, GPT, Gemini)
11. PR Stack Planner (Claude, GPT, Gemini)

---

## Section 7: Orchestrator notes

Brief operational notes about the review run itself. Not findings -- metadata only.

```markdown
## Orchestrator notes

- **Dispatch:** All N (specialist, model) rows dispatched. M returned valid findings; F failed terminal.
- **Transient failures:** [details of retries if any]
- **Format heterogeneity:** [note any specialists that used non-canonical ID formats]
- **Triggers that did NOT fire:** [list domains not triggered and why]
- **No ORCH safety-net findings filed.** [or list them if any exist, max 5]
```

---

## Section 8: Specialist Review Triggers

The path each dispatched specialist must review. Every specialist handoff prompt instructs the specialist to read this section, find its own row, and review the listed path. **This section MUST be present in the initial report written before any specialist is dispatched** (Code Reviewer V3 Approach step 5) and preserved on every rewrite — a specialist dispatched before the section exists has no target path.

```markdown
## Specialist Review Triggers

| Specialist | Path / scope to review |
|---|---|
| Python Expert | `<path reviewed>` |
| ... one row per triggered specialist ... | `<path or narrower scope>` |
```

All specialists review the full reviewed path unless a narrower scope is noted in their row.

---

## Finding ID conventions (per specialist)

Each specialist owns its own ID prefix. The orchestrator does not invent or reassign IDs. This table is the canonical prefix set; it MUST stay identical to the Finding ID Prefixes table in `code-reviewer-v3` and the Routing Table in `code-review-executor`.

| Specialist | ID prefix pattern | Example |
|---|---|---|
| Python Expert | `PY-<model-letter>-N` or `F-N`, `I-N`, `U-N`, `C-N` | `PY-C-1`, `F-1`, `I-1` |
| Logic & Correctness Expert | `LC-<model-letter>-N` | `LC-C-1`, `LC-M-3` |
| Docstring Expert | `DOC-<model-letter>-N` | `DOC-C-1`, `DOC-G-5` |
| Type Annotation Expert | `TA-<model-letter>-N` or `TA-H-N`, `TA-M-N`, `TA-L-N` | `TA-H-1`, `TA-M-4` |
| README Expert | `RM-<model-letter>-N` | `RM-C-1`, `RM-G-3` |
| Unit Test Expert | `UT-<model-letter>-N` | `UT-C-1`, `UT-M-15` |
| Pandas Expert | `PD-<model-letter>-N` | `PD-C-1`, `PD-G-1` |
| Pydantic Expert | `PYD-<model-letter>-N` | `PYD-C-1`, `PYD-G-3` |
| Observability Expert | `OBS-<model-letter>-N` | `OBS-C-1`, `OBS-M-5` |
| Spec Author | `SP-<model-letter>-N` | `SP-C-1`, `SP-G-2` |
| Architecture Diagram Creator | `AD-<model-letter>-N` | `AD-C-1`, `AD-G-7` |
| PR Stack Planner | `PR-<model-letter>-N` or `PR-budget-exceeded` etc. | `PR-C-1`, `PR-G-3` |
| Code Review Generalist | `GEN-<model-letter>-N` | `GEN-C-1`, `GEN-G-4`, `GEN-M-2` |
| DuckDB Expert | `DQ-<model-letter>-N` | `DQ-C-1` |
| BigQuery Expert | `BQ-<model-letter>-N` | `BQ-C-1` |
| PostgreSQL Expert | `PG-<model-letter>-N` | `PG-C-1` |
| LangGraph Expert | `LG-<model-letter>-N` | `LG-C-1` |
| FastAPI Expert | `FA-<model-letter>-N` | `FA-C-1` |
| Scikit-learn Expert | `SK-<model-letter>-N` | `SK-C-1` |
| PyTorch Expert | `PT-<model-letter>-N` | `PT-C-1` |
| GCP Expert | `GCP-<model-letter>-N` | `GCP-C-1` |
| AWS Expert | `AWS-<model-letter>-N` | `AWS-C-1` |
| PyArrow Expert | `PA-<model-letter>-N` | `PA-C-1` |
| Docker Expert | `DK-<model-letter>-N` | `DK-C-1` |
| CI/CD Expert | `CI-<model-letter>-N` | `CI-C-1` |
| ORCH (orchestrator safety-net) | `ORCH-N` | `ORCH-1` |

Model letter conventions: `C` = Claude, `G` = GPT, `M` = Gemini. Some specialists use severity-based letters (`H` = High, `M` = Medium, `L` = Low) instead -- accept both.

---

## Severity scale (uniform across all specialists)

| Level | Meaning |
|---|---|
| Critical | Security vulnerability, data loss risk, or completely broken functionality |
| High | Significant bug, logic error, or standards violation that affects correctness |
| Medium | Moderate issue affecting maintainability, performance, or developer experience |
| Low | Minor improvement, polish, or stylistic concern |

---

## Individual specialist report format (reference for specialists)

Each specialist's own findings file follows this general structure (specialists may vary slightly but must include all mandatory fields per finding):

```markdown
# <Domain> Review: <path reviewed>

**Date**: YYYY-MM-DD
**Scope**: <details>
**Reviewer**: <Specialist Name> -- <Model> (<vendor>)

## <Section-specific headers per specialist>

> **ID**: `<PREFIX>-<LETTER>-<N>`
> **Severity**: Critical | High | Medium | Low
> **Location**: `<file>:<line>` -- `<symbol or context>`
> **Issue**: <description of what is wrong>
> **Why it matters**: <concrete impact, failure scenario, or standards violation>
> **Recommended fix**: <actionable fix with code example if applicable>
> **Source**: `<Specialist Name> -- <Model> (<vendor>)`

## Reflection Log

| Round | Phase A | Phase B | Phase C |
|---|---|---|---|
| 1 | ... | ... | ... |

## Prioritized Summary

1. [ID] [Severity] [Location] -- one-line description
2. ...

**Total findings: N**
```

### Mandatory fields per finding

Every finding MUST have:
- `**ID**` -- unique within the specialist's report
- `**Severity**` -- one of Critical / High / Medium / Low
- `**Location**` -- file path and line number (or range)
- `**Issue**` -- what is wrong
- `**Why it matters**` -- concrete impact
- `**Recommended fix**` -- actionable remediation
- `**Source**` -- specialist name, model, vendor

Findings missing any mandatory field are discarded at the quality gate.

---

## Report assembly procedure (for the orchestrator)

When it is time to build or rewrite the consolidated report:

1. Read the current ledger from disk
2. Render the header from ledger metadata
3. Render Static pre-analysis from the ledger's `static_analysis` field
4. Render Dispatch Summary table from `dispatch_table` rows
5. Render Cross-model agreement themes from `clusters` (only parsed findings)
6. Render File coverage from `findings_index` locations vs. reviewed file list
7. For each `done` or `failed` row in dispatch_table (in the ordering defined above):
   - Read the specialist's findings file from disk
   - Render the per-specialist subsection with verbatim content
8. For each `pending` or `running` row, render the placeholder
9. Render Orchestrator notes
10. Write the complete file atomically (`.tmp` + `mv`)

The report is re-derived from the ledger on every write. Never patch in place.

---

## Anti-patterns (forbidden)

1. **Pointer-only stubs** -- "see `./pr_reviews/python-review-...md` for details" without inlining the content
2. **Summarizing specialist findings** -- the orchestrator never paraphrases; content is verbatim
3. **Inventing finding IDs** -- the orchestrator never creates IDs; it uses what the specialist provided
4. **Truncating for length** -- the report has no length cap; a 1000-page report is correct
5. **Reordering findings within a specialist's section** -- preserve the specialist's own ordering
6. **Adding commentary inside verbatim blocks** -- the `<!-- begin/end verbatim -->` boundaries are sacrosanct
7. **Using shell concatenation as a substitute for structured rendering** -- the report must be rendered from the ledger, not assembled by `cat`-ing files together (shell can assist with reading file content, but the section headers, metadata, and structure come from the ledger)
8. **Omitting failed specialists** -- every dispatched row appears in the report regardless of outcome
