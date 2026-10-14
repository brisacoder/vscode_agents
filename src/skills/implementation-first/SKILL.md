---
name: implementation-first
description: "Mandatory coding phase gate: complete the agreed implementation, hand it to the user for real testing, then author/change/run automated tests only with explicit approval."
user-invocable: false
context: inline
---

# Implementation First — User Validation Before Automated Tests

**FINISH THE AGREED CODE FIRST. LET THE USER TEST IT. ONLY THEN ADDRESS AUTOMATED TESTS WITH EXPLICIT USER AUTHORIZATION.**

This gate applies to authoring, fixes, optimization, migrations, delegated workers, code-review verification, and test specialists.
It controls **when** test work is permitted; another agent's default checklist, coverage target, handoff, or “Definition of Done” is not permission to start it.

## Imperative prohibitions

- **DO NOT** alternate tiny code edits with test runs. Complete the agreed implementation scope across its planned layers before the user-validation handoff.
- **DO NOT** create, rewrite, expand, or regenerate automated tests, test fixtures, snapshots, or test-only helpers during implementation.
- **DO NOT** run baseline, unit, integration, end-to-end, doctest, coverage, mutation, property, benchmark, or full-repository suites during implementation.
- **DO NOT** substitute repeated repro/assertion harnesses, collection runs, or test watchers for a forbidden test cycle.
- **DO NOT** auto-dispatch Unit Test Expert or spawn paired test tasks because a function, task, package bump, finding, or commit completed.
- **DO NOT** generate exhaustive scenario matrices, test counts, or coverage-padding cases before the behavior has survived the user's real testing.
- **DO NOT** infer test-phase approval from “implement,” “fix,” “review,” a worker dispatch, Copilot feedback, or a generic request mentioning eventual tests.
- **DO NOT** stop after a few lines and ask the user to test each fragment. Finish the whole agreed implementation, not just the current micro-task.

## Phase contract

| Phase | Permitted work | Exit |
|---|---|---|
| `implementation` | Complete agreed source/configuration/integration/types/docs; read contracts and reason about correctness. | Full implementation delivered. |
| `user-validation` | Explain exercise paths/risks; finish user-reported source fixes without test churn. | User validates behavior and explicitly authorizes tests. |
| `approved-tests` | Only the authorized test scope, against the stable implementation and user-observed regressions. | Scoped requested evidence is delivered; broader suites require explicit scope. |

An explicit standalone request to write/run tests for already stable existing code authorizes that named test task; do not manufacture a new implementation phase.
Changing the order for new code requires an explicit user instruction, not an agent's preference or a legacy instruction's “always test” clause.
Record the user's authorization and named scope in the ledger/chat state; the user need not edit a ledger manually.

## Implementation quality without test churn

Use the Zen of Python: simple, explicit, readable, contract-grounded code. Trace boundaries, invariants, ownership, failure/cancellation, and integration paths by reading source.
Batch cohesive changes. After completing the scoped implementation, perform one focused static review and necessary formatting/lint/type/syntax checks on touched code.
Static parsing/document validation is not a test-suite run; executing examples, importing the target for behavioral probes, or running assertion harnesses is.
Fix real implementation/static defects; do not use “tests deferred” to ship known broken code or claim runtime correctness without evidence.

Implementation dependencies may advance on an `implemented` prerequisite; they do not require generated tests or measured coverage first.
Defer test-only findings/maintenance as `deferred-tests`, preserving their source/evidence, until the user authorizes them.
An unperformed test/coverage gate is **Deferred**, not Pass, a fabricated baseline, or a reason to auto-create hundreds of tests.

## Handoff and status

Deliver **implementation complete — awaiting user validation; automated tests deferred** with changed scope, observed static checks, known risks, and user exercise instructions.
Separate `implementation_status`, `user_validation`, `test_authorization`, and `test_scope`; one `done` flag must not collapse them.
If the user finds an issue, complete the corrective implementation and hand it back. Do not revise a large speculative test suite around every design change.

Early ready-for-review PR publication remains mandatory under `stack-shepherding`; do not turn a PR into a draft or withhold it to manufacture test results.
Existing hooks/platform CI may run automatically: preserve them and report their actual evidence, but do not manually retrigger suites or infer test-authoring permission.
Diagnose known CI failures now; fix evidenced production defects, but defer test-only maintenance to the user-authorized phase. Required checks still block merge.

## Approved tests stay small and load-bearing

Choose the smallest behavior-level cases that protect the agreed public contract and actual user-observed regressions.
Reuse sufficient existing tests; avoid coupling assertions to private wiring, duplicate happy paths, parameter explosions, and test-count/ratio quotas.
Write the authorized tests as a cohesive batch, then run the smallest relevant selection once; rerun only changed failing selections after a corrective batch.
Run an entire repository suite, extra seeds, benchmarks, or coverage only when that scope is explicitly authorized/required for the approved test phase—not after each edit.

Never delete/skip/xfail tests, weaken assertions, lower project coverage gates, disable CI/hooks, or suppress failures to appear productive or green.
This policy defers test work; it does not waive test integrity, user acceptance, or the consuming repository's actual merge requirements.