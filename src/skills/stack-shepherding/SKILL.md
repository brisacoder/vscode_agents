---
name: stack-shepherding
description: "PR lifecycle: all Copilot feedback, reply-and-resolve, immediate CI-failure action, and first-commit ready PRs. Commands live in github-stacking."
user-invocable: false
context: inline
---

# Stack Shepherding — Review Early, Address Everything

**Address ALL Copilot comments and advisories. For every addressed review conversation: post its reply, RESOLVE the conversation, and verify resolution.**
**Known CI failures are actionable NOW, even when the workflow or other jobs are still running.**
**Every PR is published ready for review immediately after its first content-bearing commit. No drafts, no “submit everything at the end” exception.**

These obligations bind the author, publisher, resolver, and watcher across the entire stack. A longer agent body cannot waive them.
Load `github-stacking` for exact commands and its bundled cookbook/API recipes; do not put command syntax or stack-shape rules in this policy.

Load `implementation-first` before delegating code. **DO NOT author/change/run automated tests during implementation; finish all agreed code, let the user test, then await approval.**
Known CI failures are still triaged immediately; fix evidenced production defects, defer test-only maintenance, and preserve actual merge requirements and feedback obligations.

## 1. Address, reply, resolve, verify

Enumerate feedback on **every PR**, including pre-existing, outdated, and newly posted feedback:

- Copilot line comments, suggested changes, review summaries, advisory lists, and check annotations/output.
- Human and other automated reviewers' comments on the same surfaces.
- Every actionable point in a summary, including points with no corresponding inline thread.

Author identity, advisory severity, an outdated diff, and absence of a blocking review do not excuse ignoring an item.
Read full paginated results. Do not treat startup cursors as permission to skip historical feedback; do not classify your own replies as new reviewer requests.

Each distinct item gets a ledger entry and its own answer:

| Disposition | Required evidence/reply | Review-thread action |
|---|---|---|
| `fixed` | Verify the correction, publish it on the owning branch, explain what changed, and link the published commit. | Resolve after the reply. |
| `already-correct` | Cite the existing behavior, guard, test, or requirement that disproves the concern. | Resolve after the reply. |
| `wont-fix` | Give an explicit, justified decision within the agent's authority; obtain the user's decision when needed. | Resolve after the decision and reply. |
| `deferred` | Explain the authorized scope decision and name the tracked follow-up issue. | Resolve after the decision and reply. |
| `awaiting-user` | State the precise question/blocker and required owner. | Leave open; this is blocked work, not completion. |

The lifecycle controller owns closure. A code worker may return the fix SHA and explanation; the controller must still **reply AND resolve**.
A local fix, a pushed commit, a reply alone, or an “all items fixed” roll-up is not a completed review conversation.
Resolve only after **all items in that thread** are addressed; never resolve a thread containing an outstanding question or an unverified promise.
Do not wait for the entire distributed CI run to finish before replying/resolving a verified published fix; continue tracking its CI separately.

After resolution, read the thread back and record its ID and resolved state. If reply or resolution fails, keep that step pending and retry without duplicating a posted reply.
If permissions prevent resolution, surface `blocked: thread-resolution-permission`; never claim the thread is resolved or the PR is done.
If a thread reopens or receives new feedback, reopen the ledger obligation and process it again.

Issue comments, review summaries, and annotations do not necessarily have resolvable threads. Answer each point with a source-linked reply and record `resolution: not applicable`.
Never invent a thread ID or pretend that an ordinary PR comment has a resolved flag.

## 2. Failed work takes precedence over running work

At startup and on every observation, inspect current-head checks/statuses, workflow runs, **individual jobs/steps**, annotations, and provider diagnostics across the stack.
Aggregate `queued`/`in_progress` status never overrides an already failed job, failed step, failure annotation, or external status.

For example: **10 failed jobs + 20 running jobs = 10 failures to triage now**, not “CI is still running; wait.”

1. Record each distinct failure with branch, head SHA, run/attempt, job/check ID, and available error evidence.
2. Begin diagnosis and route the smallest corrective action in the same observation cycle. Fix shared causes on the lowest owning branch, then propagate upward.
3. Preserve the rest of the failure queue. A worker push invalidates stale context, not the obligation to re-evaluate remaining failures against the new head.
4. Use available job logs, annotations, failed steps, or provider links. An unavailable whole-run log is an evidence-fetch limitation, not a reason to ignore known failures.
5. Wait/back off only after actionable failures and feedback have been routed, or a specific blocker has been recorded. Never wait merely because another job is running.

Distinguish failed, still-running, missing, skipped, and superseded checks. Compare current attempts and SHAs; old failures are historical evidence, not proof the new head fails.
Re-runs are not code fixes. Retry only supported transient/infrastructure failures or missing triggers; do not repeatedly rerun a deterministic defect.
Use a bounded attempt ledger and escalate genuine permission/infrastructure/decision blockers. A failed API fetch is unknown coverage, not green CI.

## 3. First commit means a ready-for-review PR

After a branch's first content-bearing commit, publish its PR **before the next authoring task or stack entry**, and verify it is open and **not draft**.
Do not wait for implementation completion, the final stack, local/remote CI, final coverage, documentation, or a human reminder to open it.
Keep first commits small and coherent; subsequent fixes/tests/docs stay on that same PR. Existing quality gates still govern completion and merging, not visibility for early review.

Confirm the branch has a real PR number/URL, correct head/base, and correct remote stack membership/order once two or more PRs exist.
An empty planned branch may temporarily lack a PR **only before its first content-bearing commit**; keep unstarted entries in the plan rather than pre-creating a branch-only stack.
A branch-only row after a commit is a publication failure: repair it immediately. A successful push or CLI exit is not proof that a PR was created.

Request/verify Copilot review through the available supported integration as soon as the PR opens. Request a fresh review after meaningful updates when the repository does not do so automatically.
Do not assume every push automatically starts Copilot, or submit duplicate requests while a review is pending. Surface unavailable review permissions/capabilities explicitly.
Convert an unexpectedly draft PR to ready immediately; never use draft status to pause feedback processing or CI remediation.

## 4. Track the whole stack, not one snapshot

Persist per-branch publication/head state, feedback dispositions, reply URLs, thread IDs/resolution, CI failures, attempts, and genuine blockers.
Record an observed successful remote postcondition before completing a ledger step; reconcile after restart rather than trusting cursors or a worker's self-report.
Re-enumerate feedback and CI after every publish, review, structural change, or merge. Newly discovered advisories remain work even when earlier threads are resolved.
Keep surviving branches current with their parents after lower merges; verify publication, approvals, checks, and thread state again on the new heads.

Every PR carries the actual plain-ASCII Jira key required by the consuming workspace. Never invent one; missing metadata blocks merge, not first-commit review publication.
Investigate a missing required workflow using its actual trigger/provider. Do not fabricate passing results, change required checks, or blindly create empty commits to hide the missing trigger.

## 5. Review-ready is not merge-ready

Early ready-for-review publication is mandatory. It does **not** authorize merging unfinished work or bypassing approval, test, security, or conversation requirements.
Merge/queue only the explicitly authorized, completed bottom-up range whose current heads satisfy the consuming repository's actual requirements.
The command expert owns the supported native merge/queue procedure; unsupported auto-merge is a capability limit, not a defect to retry forever.
If the caller only authorized review/repair, report merge-readiness and hand off to an authorized merger; do not expand authority yourself.

## Completion and blocked handoff

For a merge-authorized shepherding run, completion requires a fresh reconciliation showing:

- Every PR is actually merged, or explicitly closed with an authorized disposition; no committed branch is left without its PR.
- Every Copilot/reviewer item has a verified disposition and its own reply; every addressed resolvable conversation is verified resolved.
- No `awaiting-user`, unresolved permission failure, unhandled advisory, or current actionable CI failure is hidden as “done.”
- All owned workers/operations have finished, and any surviving stack state has been reconciled.

For review/repair-only work, report **review/repair complete; awaiting authorized merge**, not “stack merged” or an unattended watcher that does not exist.
In a real background runner, continue monitoring while authorized work remains. In a one-shot session, perform the useful current cycle and persist an explicit handoff/blocker.
Green CI, a reply count, queued status, an approval, or a clean cascade alone is not terminal completion.

## Ownership boundary

- This skill owns **what must happen, when, and what counts as completion**.
- `github-stacking` owns **verified native CLI/API recipes, version/capability checks, and recovery mechanics**.
- PR Stack Planner owns **stack decomposition, diff/file budgets, formatting, tests, and merge-readiness quality gates**.
