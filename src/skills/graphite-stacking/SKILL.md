---
name: graphite-stacking
description: Canonical Graphite CLI (gt) command set and stacked-PR workflow for this workspace. Load whenever an agent creates branches, commits, opens or updates pull requests, restacks, syncs, submits, or monitors PRs. In this workspace stacked PRs are the default unit of delivery, not a special case — a unit of work is decomposed into a stack of small, dependent branches (one PR per branch) that are planned first, built bottom-up, and submitted and monitored as a stack with `gt submit --stack`. Replaces ad-hoc `git checkout -b` / `git push` / `gh pr create` flows. Covers gt init/create/modify/restack/sync/submit/log/checkout/up/down/move/track, the plan-stack-first discipline, and the restack-cascade hazards (let a cascade settle before reacting; a restack force-push does not reliably re-fire org-level/required workflows, leaving PRs stuck on absent checks). Apply for any Plan, Enforce, Author, Fix, Watch, or Resolve step that touches branches or PRs.
user-invocable: false
context: fork
---

# Graphite Stacking — Canonical Workflow

This workspace ships changes as **stacks of pull requests** built and managed with the **Graphite CLI (`gt`)**. A stack is a chain of small, dependent branches — each branch is one PR, each PR builds on the one below it, and the whole chain is submitted and reviewed together. Stacking is the **default**, not an advanced option: any unit of work large enough to be more than one cohesive change is a stack.

`gt` wraps `git` and GitHub: it tracks parent/child relationships between branches, restacks descendants automatically when a lower branch changes, and creates/updates one PR per branch with the correct base. Do not hand-roll stacks with `git checkout -b` + `git push` + `gh pr create` — that loses the parent metadata `gt` needs to restack and to set PR bases correctly.

## Stacks are first-class

- **The unit of delivery is the stack, not the single PR.** Plan the stack before writing code (see *Plan the stack first*). Each branch in the stack is a reviewable, independently-mergeable-in-order PR.
- **One logical change per branch.** Each branch/PR does one thing and leaves the build green. Lower branches never depend on higher ones.
- **Bottom-up.** Branch 1 sits on trunk; branch 2 sits on branch 1; and so on. Reviewers read and merge from the bottom up.
- **Submit and monitor as a stack.** `gt submit --stack` opens/updates every PR in the chain at once and keeps their bases correct. Monitoring tracks the whole stack's CI and review state, not one PR in isolation.

## Plan the stack first

Before any code is written, decompose the work into an ordered stack and write the plan down (issue checklist, PR description, or a markdown file under `docs/plan/`). The plan is the contract; each branch cites the stack entry it implements.

For each branch in the stack, the plan names:

1. **Branch name** — `gt`-friendly, conventional-commit-flavored (e.g. `feat-order-model`, `feat-checkout-service`).
2. **Position** — its index in the stack and its parent branch (branch 1 → trunk; branch N → branch N-1).
3. **Scope** — the files/symbols it adds or changes. Each branch's primary file set is disjoint from its siblings except trivial integration glue.
4. **Behavior gate** — what works after this branch merges; every branch leaves build, tests, and lint green.
5. **Tests** — every branch ships its own tests so each PR independently meets the workspace 75% coverage gate. No "tests branch" at the top of the stack.

A stack branch should stay well under the workspace per-PR size cap; if a single branch would exceed it, split it into two stacked branches.

## Setup (once per repo / once per checkout)

- **Initialize Graphite on the repo (choose trunk)**: `gt init --trunk main` (omit `--trunk` to pick interactively). Re-run to change trunk.
- **Authenticate for PR creation**: `gt auth --token <token>` (token from the Graphite dashboard). Required before `gt submit` can open PRs.
- **Adopt an existing branch into a stack**: `gt track --parent <parent-branch>` from the branch you want tracked.

## Create and build the stack (bottom-up)

- **Start the bottom branch on trunk**: from trunk, stage changes and run `gt create <branch-name> -m "feat(scope): ..."`. `gt create` makes a new branch stacked on the current one and commits the staged changes.
  - Stage-all variant: `gt create <branch-name> -a -m "..."`.
- **Stack the next branch on top**: with the previous branch checked out, `gt create <next-branch> -m "..."`. It is automatically parented to the current branch.
- **Amend the current branch's commit** (after edits): `gt modify -a` (amend) or `gt modify -c -m "..."` (new commit on the branch). `gt modify` automatically restacks all descendants.
- **Insert a branch into the middle of a stack**: `gt create <branch> --insert`.

Never use `git commit` + `git checkout -b` to extend a stack; use `gt create` / `gt modify` so parent metadata and restacking stay correct.

## Keep the stack consistent

- **Restack after history changes**: `gt restack` rebases each branch onto its parent so the chain stays linear. `gt` runs this automatically after `gt modify`, but run it explicitly after manual `git` operations.
- **Sync with remote and clean merged branches**: `gt sync` fast-forwards trunk, restacks what it can, and prompts to delete branches whose PRs merged/closed. Run it at the start of a session and after parts of a stack merge.
  - Non-interactive cleanup: `gt sync --no-interactive --force` (use with care).
- **Pull one branch/PR from remote**: `gt get <branch-or-PR>`.
- **Resolve a restack/sync conflict**: resolve in the working tree, then `gt continue` (or `gt continue -a`). Abort with `gt abort`.

## Navigate the stack

- `gt log` (alias `gt ls`) — show the stack and parent/child relationships. `gt log --stack` limits to the current stack.
- `gt checkout <branch>` — switch branches (interactive if no name). `gt up` / `gt down` — move one branch toward the tip / toward trunk. `gt top` / `gt bottom` — jump to the ends. `gt parent` / `gt children` — inspect relationships.

## Submit the stack

- **Submit the whole stack** (create/update one PR per branch, correct bases): `gt submit --stack` (alias `gt ss`). This restacks-validates, force-pushes with lease, and opens/updates every PR from trunk to the tip.
  - **Draft the whole stack**: `gt submit --stack --draft`.
  - **Update only existing PRs** (no new ones): `gt submit --stack --update-only`.
  - **Preview without pushing**: `gt submit --stack --dry-run`.
  - **Non-interactive** (CI / agent use): add `--no-edit` (keep existing metadata) or `--no-interactive`; supply reviewers with `-r <user1,user2>`.
- **Submit narrowly** (just the current branch and its ancestors): `gt submit` (without `--stack`).
- After parts of the stack merge, run `gt sync` then `gt submit --stack` again to rebase the survivors onto the new trunk and refresh their PRs.

## Monitor the stack

- **CI / merge state per branch**: the stack is `gt log`; for each branch's PR use `gh pr view <branch> --json state,statusCheckRollup,mergeStateStatus,reviewDecision,autoMergeRequest` and `gh pr checks <branch>`. Open the stack page with `gt pr --stack`.
- A stack is green only when **every** PR in it is green and its base is correct. A failure on a lower branch blocks every branch above it — fix the lowest failing branch first, then `gt sync` / `gt submit --stack` to propagate.

## Drain the stack (green is not done — merged is done)

A green, mergeable stack is **not** a finished stack. A stack with a merge queue drains **bottom-up, one PR at a time**: the bottom PR enters the queue, merges, trunk advances, the survivors rebase, the next PR becomes mergeable, and so on. "Every PR is green and `mergeable`" is the state of a stack that is *ready to start draining* or *between merges* — it is **in progress**, not complete. The only terminal state is **every PR `merged` or `closed`**.

This distinction is the cause of the most common monitoring bug: an agent sees an all-green stack, concludes "nothing to do", and stops — leaving a stack that never actually merges because nothing ever fed the bottom PR into the queue, or because one PR merged and no one advanced the next.

**The draining invariant.** While *any* PR in the stack is still open, the stack is not done, and at any moment at least one open PR should be *making progress toward merge*: in the merge queue (`mergeStateStatus: QUEUED` / an `autoMergeRequest` present / GitHub's "will merge when ready"), or running/queued CI, or mid-rebase after a lower merge. If every open PR is idle — green, mergeable, but **none enqueued and none progressing** — the stack has **stalled**, and the correct action is to push the lowest open PR into the merge queue, not to exit.

**Feed the bottom open PR into the merge queue.** This repo uses GitHub's native merge queue. Enqueue the lowest still-open PR whose CI is green, whose base is trunk (or an already-merged branch), and which is `mergeable`:

```sh
gh pr merge <pr-number-or-branch> --auto --squash    # adds to the merge queue; merges when required checks pass
```

`--auto` enables auto-merge, which places the PR in the queue and merges it once required checks pass — it does **not** bypass the queue or required checks. Only the **lowest open PR** is enqueued at a time (its descendants are not mergeable until it merges and they rebase onto the new trunk). After the bottom PR merges:

1. `gt sync` — fast-forwards trunk, restacks survivors, prompts to delete the merged branch.
2. `gt submit --stack` — rebases the survivors onto the new trunk and refreshes their PRs (this is a cascade; let it settle per *Restack cascades and stuck PRs*).
3. The new bottom PR becomes `mergeable`; enqueue it next.

Repeat until the stack is empty. Monitoring continues across every merge — do not stop until the **last** PR is merged or closed.

## Restack cascades and stuck PRs (operational hazards)

Submitting or restacking a stack force-pushes **every descendant branch**, not just the one you edited. This produces two recurring failure modes that monitoring and fixing agents must handle explicitly.

### 1. A restack/submit is a multi-branch cascade — let it settle before reacting

`gt restack` + `gt submit --stack` rewrites and force-pushes branch N and **all** branches above it in one operation. While that cascade is in flight:

- Each pushed branch advances its head SHA; GitHub tears down the old check runs and (usually) starts new ones a few seconds later. There is a window where a branch has a **new head SHA but no check runs yet** — this is normal mid-cascade, not a failure.
- A monitor that polls during this window sees "no checks", "BEHIND", or transient `mergeable: null` on several branches at once and will **spam-retry / re-dispatch fixes** for problems that are merely the cascade in progress.

**Rule:** treat a known in-flight `gt submit --stack` / `gt restack` as a quiet period. The submitting agent records when the cascade starts and finishes; a monitor must **quiesce** (pause polling, or sleep a fixed settle interval — at least 60–90s) until the cascade completes and every branch's head SHA is stable, before classifying any branch as failed/stuck. Do not open fix work against a branch whose SHA is still moving. (A background monitor can be paused with `SIGSTOP` and resumed with `SIGCONT` so it does not spam-retry while a cascade is in flight.)

### 2. A Graphite restack force-push does NOT reliably re-fire org-level / required workflows

This is the lesson that bites hardest. A lease-protected force-push from `gt submit` / `gt restack` updates the branch ref, but **GitHub does not always dispatch the org-level, required, or `workflow_run`/`on: push`-filtered workflows** for the new SHA — especially for branches that were only *rebased* (no content delta) by the cascade. The PR then sits **stuck**: `mergeStateStatus` stays `BLOCKED` / `UNSTABLE` waiting on a required check that is **absent**, not failed, and never arrives. Polling forever will never turn it green.

How to recognize it (distinct from a real CI failure):

- The required/expected check is **missing entirely** from `statusCheckRollup` for the current head SHA (not present-and-`failure`, not present-and-`queued`/`in_progress`).
- The branch was rebased by a restack with no real content change, yet its checks did not restart.
- Lower branches in the same stack got their checks but a rebased upper branch did not.

How to recover (escalating, least-invasive first):

1. **Re-dispatch the workflow** for the head SHA: `gh workflow run <workflow> --ref <branch>` (if it accepts `workflow_dispatch`), or `gh run rerun <run-id>` for the last run on that branch.
2. **Nudge the ref with an empty commit** so an `on: push` workflow fires for a genuinely new SHA: `gt modify -c -m "chore: re-trigger CI"` on the branch (Graphite restacks descendants), then `gt submit --stack`. Prefer this over a raw `git commit --allow-empty` so stack metadata stays intact.
3. **Re-submit the stack** (`gt sync` → `gt restack` → `gt submit --stack`) if trunk moved underneath it.

A stuck-on-absent-check PR is **not** a `ci-failure-*` (nothing failed) and **not** a `merge-conflict` (nothing conflicts) — it is a *missing-required-check* condition with its own recovery. Classify and route it as such rather than retrying the same poll or re-running a fix that already landed.

## Hard rules

1. **Use `gt`, not raw git, for anything that creates or moves branches or opens/updates PRs.** Raw `git checkout -b`, `git push`, and `gh pr create` lose stack metadata. Plain `git add`/`git status`/`git log`/`git diff` for inspection are fine.
2. **Plan the stack before writing code.** An unwritten plan is no plan. Build bottom-up against the plan.
3. **One logical change per branch; each branch leaves the build green and ships its own tests.**
4. **Submit and monitor the whole stack** (`gt submit --stack`), not one PR at a time.
5. **Never force-push raw** (`git push --force`). `gt submit` already force-pushes with lease; do not bypass it.
6. **After any lower branch changes, restack** (`gt restack` / `gt modify` does it automatically) so descendants stay correct before re-submitting.
7. **A restack/submit is a multi-branch cascade — let it settle before reacting, and never assume a force-push re-fired CI.** Quiesce monitoring during an in-flight `gt submit --stack` / `gt restack`; a PR stuck on a *missing* (absent, never-fired) required check is a re-trigger condition, not a CI failure or conflict (see *Restack cascades and stuck PRs*).
8. **Green is not done — merged is done.** A stack is complete only when **every** PR is `merged` or `closed`. While any PR is open, keep driving and monitoring: at least one open PR must always be progressing toward merge (enqueued, CI running, or rebasing). A green, `mergeable` stack with nothing in the merge queue has **stalled** — feed the lowest open PR into the queue (`gh pr merge <pr> --auto --squash`), then `gt sync` + `gt submit --stack` after each merge and enqueue the next. Never treat all-green as a stop condition (see *Drain the stack*).
