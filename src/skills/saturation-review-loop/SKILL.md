---
name: saturation-review-loop
description: The canonical three-round, three-phase saturation loop that every review-mode specialist agent uses to drive its findings to zero-delta closure. Load whenever the agent is in Review mode (auditing a path, a module, a package, a graph, a Dockerfile, a workflow, or any other artifact) and needs to produce a complete, idempotent findings report whose contents do not change on re-runs. The loop is the named anti-pattern cure for "the next reviewer finds more" — Phase A (Verify) re-reads existing findings in parallel and renders Confirmed / Improved / Disproved; Phase B (Hunt) launches diverse-prior hunter subagents that draft independently and only dedupe at the end; Phase C (Pattern propagation) takes every new finding and searches sibling sites for the same anti-pattern. Three rounds max; the loop terminates on the first zero-delta round and records why. Each agent supplies its own section IDs (F/I/A/P/C/S/L/U/PY for Python Expert, LC.atomicity/invariants/check-then-act/idempotency/boundary for Logic and Correctness, FA.deps/async/response/middleware/security/background/routing for FastAPI, BQ-Heresy/security/fundamentals for BigQuery, etc.) and its own hunter persona roster (the Pessimist, the Adversary, the Scaler, the Maintainer, the Beginner, the Pythonista for Python; the Corruptor, the Racer, the Retrier, the Edge Finder for Logic and Correctness; the Reader, the Skeptic, the Literalist for the checklist-free Code Review Generalist; etc.), but the loop mechanics are uniform — do not paraphrase them and do not weaken them.
user-invocable: false
context: fork
---

# Saturation Loop — Canonical Review Mechanics

This is the binding loop every Review-mode specialist runs. The loop is the cure for the historical failure mode where running the same review a second time finds more issues than the first run. After the loop terminates, re-running the agent on the same input produces the same report.

Each agent supplies two inputs of its own:

1. **Section IDs** — the agent's review categories (e.g. `F`, `I`, `A`, `P`, `C`, `S`, `L`, `U`, `PY` for Python Expert; `LC.atomicity`, `LC.invariants`, `LC.check-then-act`, `LC.idempotency`, `LC.boundary` for Logic and Correctness; `FA.deps`, `FA.async`, `FA.response`, `FA.middleware`, `FA.security`, `FA.background`, `FA.routing` for FastAPI; etc.).
2. **Hunter personas** — the diverse-prior reviewers (e.g. The Pessimist, The Adversary, The Scaler, The Maintainer, The Beginner, The Pythonista; or The Corruptor, The Racer, The Retrier, The Edge Finder; or The Reader, The Skeptic, The Literalist for the checklist-free generalist; or whatever the agent has defined).

Everything else is the same in every agent.

## Round structure

Each round has three phases. The loop terminates on the first zero-delta round or after **three** rounds — whichever comes first. The termination reason MUST be stated in the Reflection Log (`terminated round N — zero-delta` or `terminated round 3 — cap reached`).

### Phase A — Verify (per round)

Launch **independent verifier subagents in parallel**, partitioned across the agent's review sections. Each verifier receives **only the findings for its assigned sections and the source code** — it does not see prior reasoning, and it does not see other verifiers' findings.

Each verifier renders a per-finding verdict from this fixed set:

- **Confirmed** — the finding is real and the description is accurate as filed.
- **Improved** — the finding is real but its description, location, severity, or recommended fix should change. State exactly what changed.
- **Disproved** — the finding is wrong (false positive, upstream guard, impossibility, misread of framework semantics, etc.). The finding is removed from the report and a one-line reason goes in the Reflection Log.

Only **Confirmed** and **Improved** findings survive to the final report. Disproved findings are deleted, not silently hidden.

Verifier-partition rules:

- Every section the agent reviews MUST be covered by exactly one verifier subagent. No section may be elided.
- Partitioning is by section, not by file. A verifier owns its assigned sections across every file in the reviewed path.
- The agent's own definition specifies which verifier owns which sections — the loop does not prescribe a partition count.

### Phase B — Hunt with diverse priors (per round)

Launch **all of the agent's hunter persona subagents in parallel**. The number and identity of hunters comes from the agent's own roster; the loop does not prescribe how many. The hunters share two binding properties no matter how many there are:

- **Diverse priors** — each hunter operates from a distinct mental model (e.g. "what could break", "what could be exploited", "what would mislead a new contributor", "what is non-idiomatic"). Two hunters with overlapping priors are a defect — the agent must collapse them.
- **No prior-findings contamination** — every hunter has the full source, the coverage matrix, and its own checklist, but does **not** see prior findings until its own draft is complete. This is the entire point of diverse priors: a hunter that sees the existing findings first will anchor on them and surface only adjacent issues.

After each hunter produces its draft list, it is shown the existing findings and removes duplicates, leaving only **deltas** (genuinely new findings). The deltas are added to the report and tagged in the Reflection Log with the producing hunter and the round number.

Every hunter MUST produce a **checklist trace** for its assigned sections even if it finds nothing. A blank "None identified" is invalid unless the trace shows the hunter walked the section's checklist end to end. Rubber-stamping is a defect.

### Phase C — Pattern propagation (per round)

For every new finding produced in this round (whether from a Phase A Improved verdict or a Phase B hunter delta), search the rest of the codebase for the **same anti-pattern at sibling call sites**. Use the most precise primitive available:

- `search/textSearch` for literal-string patterns (e.g. `bare except:`, `os.path.join`, `requests.get(`).
- `search/usages` for symbol-level propagation (e.g. every call site of a misused helper).
- AST or grep regex when neither of the above is precise enough.

Each additional match becomes its own finding. The original is annotated with the propagation count; the new ones carry `Origin: propagation-of-<source-ID> (round N)`.

### Termination

After Phase C completes for the round, count the new findings (Phase A Improved verdicts + Phase B deltas + Phase C propagations). If the count is zero, finalize the report. Otherwise begin the next round.

Hard cap: **3 rounds**. If round 3 produces non-zero new findings, finalize anyway and record `terminated round 3 — cap reached` in the Reflection Log. The agent does not run a round 4 even if more findings are likely; the report ships with what is known and the user re-runs if they want another pass.

Record per-round counts in the Reflection Log:

```
Round counts: round 1 added X, round 2 added Y, round 3 added Z
Termination: zero-delta round N | cap reached round 3
```

## What the loop is not

- **Not a debate**. Phase A is a verdict, not a discussion. A verifier that "isn't sure" must mark the finding **Improved** with a specific clarification or **Disproved** with a specific reason. "Maybe" is not a verdict.
- **Not a re-write**. Phase B hunters file new findings; they do not edit existing ones. Existing-finding improvements come from Phase A.
- **Not a budget**. The loop terminates on zero-delta, not on time. If three rounds finish in five minutes because the input is small, that is the correct behavior. If three rounds take an hour because the input is large, that is also the correct behavior — do not skip rounds to save time.
- **Not optional**. Skipping the loop is a defect. The whole point of the design is that re-running the agent on the same code yields the same report; skipping the loop guarantees that re-running will surface findings the first run missed.

## Inputs the calling agent must provide

The agent's body MUST define, before invoking the loop:

1. **Section IDs and ownership** — which sections exist, and which verifier subagent owns which section in Phase A.
2. **Hunter roster** — the named hunter personas with their owned sections and one-line prior. The roster size is determined by the agent (Python Expert uses six; Logic and Correctness Expert uses four; the Code Review Generalist uses three lens-hunters; smaller agents use as few as two), but every hunter must have a unique prior and at least one owned section. For a checklist-free generalist the "sections" are reading angles rather than domains, and the hunters are distinguished by stance (intent vs. behavior-on-input vs. literal tokens) rather than by subject area.
3. **Pattern-propagation hint** — the agent should suggest, per section, which search primitive is most precise (literal string / symbol usage / AST). This is advisory — the loop will fall back to `search/textSearch` if nothing else is named.

The loop never invents sections or hunters. If the agent's body is silent, the loop logs a Reflection note (`section roster incomplete` / `hunter roster incomplete`) and proceeds with what is defined.

## Reflection Log — what the loop writes

The loop appends the following to the report's Reflection Log section:

```
- Round counts: round 1 added X, round 2 added Y, round 3 added Z
- Termination: <zero-delta round N | cap reached round 3>
- Disproved: <ID> — <reason>
- Improved: <ID> — <what changed>
- Added by reflection: <ID> — section, round, persona, one-line summary
- Added by propagation: <ID> — propagated from <source ID>, round N
```

The Reflection Log is **append-only across rounds**. Earlier entries are not edited or removed — even Disproved findings from a later round leave their disproval reason in the log.
