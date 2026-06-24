---
name: graphite-stacking
description: Canonical Graphite CLI (gt) command set and stacked-PR workflow for this workspace. Load whenever an agent creates branches, commits, opens or updates pull requests, restacks, syncs, submits, or monitors PRs. In this workspace stacked PRs are the default unit of delivery, not a special case — a unit of work is decomposed into a stack of small, dependent branches (one PR per branch) that are planned first, built bottom-up, and submitted and monitored as a stack with `gt submit --stack`. Replaces ad-hoc `git checkout -b` / `git push` / `gh pr create` flows. Covers gt init/create/modify/restack/sync/submit/log/checkout/up/down/move/track and the plan-stack-first discipline. Apply for any Plan, Enforce, Author, Fix, Watch, or Resolve step that touches branches or PRs.
user-invocable: false
context: inline
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

- **CI / merge state per branch**: the stack is `gt log`; for each branch's PR use `gh pr view <branch> --json state,statusCheckRollup,mergeStateStatus,reviewDecision` and `gh pr checks <branch>`. Open the stack page with `gt pr --stack`.
- A stack is green only when **every** PR in it is green and its base is correct. A failure on a lower branch blocks every branch above it — fix the lowest failing branch first, then `gt sync` / `gt submit --stack` to propagate.

## Hard rules

1. **Use `gt`, not raw git, for anything that creates or moves branches or opens/updates PRs.** Raw `git checkout -b`, `git push`, and `gh pr create` lose stack metadata. Plain `git add`/`git status`/`git log`/`git diff` for inspection are fine.
2. **Plan the stack before writing code.** An unwritten plan is no plan. Build bottom-up against the plan.
3. **One logical change per branch; each branch leaves the build green and ships its own tests.**
4. **Submit and monitor the whole stack** (`gt submit --stack`), not one PR at a time.
5. **Never force-push raw** (`git push --force`). `gt submit` already force-pushes with lease; do not bypass it.
6. **After any lower branch changes, restack** (`gt restack` / `gt modify` does it automatically) so descendants stay correct before re-submitting.
