---
name: github-stacking
description: Canonical GitHub-native stacked pull request workflow — GitHub's own built-in stacked-PR platform feature, driven locally via the `gh stack` extension for GitHub CLI. Load whenever an agent creates branches, commits, opens or updates pull requests, rebases, syncs, submits, merges, or monitors PRs. In this workspace stacked PRs are the default unit of delivery, not a special case — a unit of work is decomposed into a stack of small, dependent branches (one PR per branch) that are planned first, built bottom-up, and submitted and monitored as a stack with `gh stack submit`. Replaces ad-hoc `git checkout -b` / `git push` / `gh pr create` flows. This workspace uses GitHub's native stacking feature exclusively — no third-party stacking CLI is used. Covers gh stack init/add/view/checkout/modify/submit/sync/rebase/push/merge/switch/up/down/top/bottom/trunk and the plan-stack-first discipline. Apply for any Plan, Enforce, Author, Fix, Watch, or Resolve step that touches branches or PRs.
user-invocable: false
context: inline
---

# GitHub Stacking — Canonical Workflow

This workspace ships changes as **stacks of pull requests** built and managed with GitHub's **native stacked pull request feature**, driven locally through the **`gh stack` extension** for GitHub CLI. A stack is a chain of small, dependent branches — each branch is one PR, each PR targets the branch of the PR below it, and the whole chain is tracked, reviewed, and merged together on GitHub itself. Stacking is the **default**, not an advanced option: any unit of work large enough to be more than one cohesive change is a stack.

`gh stack` wraps `git` and the GitHub platform: GitHub itself (not a third-party service) tracks the chain of PRs, shows a stack map in every PR's merge box, enforces branch-protection rules and required CI against the **stack's base branch** (e.g. `main`) for every layer — not just the bottom one — and understands merge queues natively. `gh stack` handles the local development loop: creating and tracking branches in dependency order, cascading rebases, pushing, and creating/linking the PRs. Do not hand-roll a stack with `git checkout -b` + `git push` + `gh pr create` — that produces an ordinary PR with no stack relationship, so GitHub will not show the stack map, will not cascade rebases, and will not apply stack-aware merge behavior.

This is a GitHub **platform** feature (currently public preview), not a third-party tool or service. Everything here is native to `git` + `gh` + github.com — no external account or token beyond the normal `gh auth login` is required.

## Stacks are first-class

- **The unit of delivery is the stack, not the single PR.** Plan the stack before writing code (see *Plan the stack first*). Each branch in the stack is a reviewable, independently-mergeable-in-order PR.
- **One logical change per branch.** Each branch/PR does one thing and leaves the build green. Lower branches never depend on higher ones.
- **Bottom-up.** Branch 1 sits on trunk; branch 2 sits on branch 1; and so on. Reviewers read and merge from the bottom up.
- **Submit and monitor as a stack.** `gh stack submit` opens/updates every PR in the chain at once and keeps their bases correct. Monitoring tracks the whole stack's CI and review state, not one PR in isolation.

## Plan the stack first

Before any code is written, decompose the work into an ordered stack and write the plan down (issue checklist, PR description, or a markdown file under `docs/plan/`). The plan is the contract; each branch cites the stack entry it implements.

For each branch in the stack, the plan names:

1. **Branch name** — `gh stack`-friendly, conventional-commit-flavored (e.g. `feat-order-model`, `feat-checkout-service`).
2. **Position** — its index in the stack and its parent branch (branch 1 → trunk; branch N → branch N-1).
3. **Scope** — the files/symbols it adds or changes. Each branch's primary file set is disjoint from its siblings except trivial integration glue.
4. **Behavior gate** — what works after this branch merges; every branch leaves build, tests, and lint green.
5. **Tests** — every branch ships its own tests so each PR independently meets the workspace's coverage gate. No "tests branch" at the top of the stack.

A stack branch should stay well under the workspace per-PR size cap; if a single branch would exceed it, split it into two stacked branches.

## Setup (once per machine / once per repo)

- **Install the CLI extension** (once per machine): `gh extension install github/gh-stack`. Requires GitHub CLI `gh` >= 2.90.0 and Git >= 2.20.
- **Install the agent skill** (once per machine, for Copilot/agent sessions to drive stacks directly): `gh skill install github/gh-stack`.
- **Authenticate** (ordinary `gh` auth — no separate stack-specific token exists): `gh auth login`.
- **Initialize a stack in the repo**: `gh stack init` (interactive — prompts for the first branch name and offers to adopt the current branch) or non-interactively `gh stack init <branch-name>` / `gh stack init <branch1> <branch2> ...` to adopt existing branches and create any missing ones. Trunk defaults to the repository's default branch; override with `gh stack init --base <branch>`.
- **Adopt branches created outside `gh stack`** (e.g. by another tool): pass their names to `gh stack init`, or use `gh stack link <branch-or-pr> <branch-or-pr> ...` to link existing branches/PRs into a GitHub-native stack without local tracking.

## Create and build the stack (bottom-up)

- **Start the bottom branch**: `gh stack init <branch-name>` (first layer on trunk).
- **Stack the next branch on top**: from the top of the current stack, `gh stack add <branch-name>` creates a new branch at HEAD and checks it out.
- **Commit work on the checked-out (top) branch** with plain `git add` / `git commit` — a stack branch is an ordinary git branch; committing to it is unremarkable.
- **One-step stage + commit + new branch**: `gh stack add -Am "<message>" <branch-name>` stages all changes (`-A`; use `-u` to stage tracked files only) and commits before creating the branch. If the current branch has no commits yet, the commit lands there instead of creating a new branch — run `gh stack view` if the resulting layering is ambiguous.
- **Insert a branch into the middle of an existing stack**: `gh stack modify`, stage an insert (`i` below the cursor, `I` above), then save with Ctrl+S. Requires a clean working tree, no rebase in progress, and no PR in the stack queued to merge.

Never use `git checkout -b` + `git push` + `gh pr create` to extend a stack — that creates a plain PR with no stack relationship (no stack map, no cascading rebase, no stack-aware merge).

## Keep the stack consistent

**A commit on a lower branch does not automatically propagate to the branches above it — cascading is an explicit step.**

- **Change a lower layer**: `gh stack down` (or `gh stack checkout <branch>`) to the branch that owns the change, commit normally, then `gh stack rebase --upstack` to cascade the change onto every branch above it, then `gh stack push` to push the updated branches.
- **Full cascading rebase** (fetches `origin` first): `gh stack rebase`. Scope with `--downstack` (trunk through the current branch) or `--upstack` (current branch through the top); skip the network fetch with `--no-trunk` (rebase stack branches onto each other only).
- **One command for the common case**: `gh stack sync [--prune]` — fetches `origin`, reconciles the remote stack (pulling down branches for PRs someone else added to the same stack), fast-forwards trunk, cascades the rebase, pushes with `--force-with-lease`, syncs each PR's state from GitHub, and links/updates the stack object on GitHub. Add `--prune` to delete local branches for merged PRs. Run at the start of a session and whenever a lower PR has merged.
- **Resolve a rebase conflict**: fix the conflict markers, `git add` the resolved files, then `gh stack rebase --continue` (or `gh stack rebase --abort` to restore every branch to its pre-rebase state).
- **Resolve a diverged stack** (local and remote stacks are not a clean prefix of each other): `gh stack sync` prompts interactively to adopt the remote stack as source of truth, delete the stack object on GitHub and recreate it with `gh stack submit`, or cancel. In a non-interactive/agent session a divergence aborts the sync without pushing or changing anything — surface this to the user rather than guessing which side should win.
- **Pull down a stack you don't have locally**: `gh stack checkout <stack-number|pr-number|pr-url|branch>`.

## Navigate the stack

- `gh stack view` (`--short` for branch names only, `--json` for machine-readable output) — every branch, its PR link, and its most recent commit.
- `gh stack checkout <branch>` — switch by name (safe non-interactively). `gh stack up [n]` / `gh stack down [n]` — move toward the tip / toward trunk. `gh stack top` / `gh stack bottom` — jump to the ends. `gh stack trunk` — jump to the stack's trunk branch.
- `gh stack switch` — interactive picker across the current stack. Requires a TTY; do not use from a non-interactive agent session.

## Submit the stack

- **Submit the whole stack** (push every branch; create a PR for every branch that doesn't have one; update PRs that already exist; link them together as a stack on GitHub): `gh stack submit`. In a non-interactive/agent session, always pass `--auto` explicitly (skip the full-screen editor, use auto-generated titles) rather than relying on TTY auto-detection.
- **Draft vs. ready for review**: with `--auto`, new PRs default to **draft**. Pass `--open` to create new PRs ready for review and mark any already-open PRs ready for review too.
- **Push without creating or editing any PR**: `gh stack push` pushes every active (non-merged, non-queued) branch in one pass with a per-branch `--force-with-lease` check; it does not touch PRs. Use this to publish commits mid-fix before you're ready to update PR descriptions.
- **No documented dry-run flag exists.** Before submitting, use `gh stack view` to confirm branch composition and `git diff --shortstat <parent>...<branch>` per branch to confirm line counts — that is the closest available preview in a non-interactive session.
- **If every PR in the stack already merged**, `gh stack submit` cannot extend that (now-complete) stack — it automatically starts a **new** stack rooted at trunk for your remaining unmerged branches. Treat that as a signal to `gh stack sync` first, not a bug.
- **Signed commits**: if the repository requires signed commits, always rebase with `gh stack rebase` (uses your local git signing configuration) and push with `gh stack push` / `gh stack submit` — never trigger the website's "Rebase stack" button from automation, because server-side rebase commits are unsigned.

## Monitor the stack

- **Per-branch CI / merge state**: `gh pr view <branch> --json state,statusCheckRollup,mergeStateStatus,reviewDecision,url` — unchanged from monitoring any ordinary PR. A workflow that triggers on `pull_request` events targeting the repo's default branch runs for **every** PR in the stack, not just the bottom one, with no workflow changes required.
- **Read a PR's stack membership/position from the API**: `gh api repos/<owner>/<repo>/pulls/<number> --jq .stack` returns `{number, size, position, base: {ref, sha}}` when the PR belongs to a stack (`null` otherwise) — useful for scripts that need "is this PR mid-stack" without running `gh stack view`.
- **A stack is green only when every PR in it is green** and every branch sits on a linear, current history relative to its parent. Merge requirements (required reviews, required checks, CODEOWNERS) are evaluated against the **stack's base branch** for every layer, so a failure or requested change on a lower branch blocks every branch above it — fix the lowest failing branch first, then `gh stack rebase --upstack` + `gh stack push` (or `gh stack sync`) to propagate the fix upward.
- **CI cost control**: workflows can read `github.event.pull_request.stack.position` / `.size` / `.base.ref` / `.base.sha` to gate expensive jobs to just the lowest-unmerged PR (`stack.base.ref == base.ref`) or the top PR (`stack.position == stack.size`) instead of running on every layer.

## Merge the stack

- **Merge the whole stack at once**: merge the top PR (on github.com, or `gh stack merge` / `gh stack merge <top-pr-number>`) — every PR below it merges too, bottom-up, in one atomic operation. If any PR in the range can't merge, none of them do.
- **Merge part of the stack**: merge a mid-stack PR (`gh stack merge <pr-number>`) — it and everything below it merge; PRs above stay open and automatically re-target the stack's base branch once their former parent is gone.
- **Merge-queue aware**: if the base branch uses a merge queue, merging (via `gh stack merge` or the website) adds the selected range to the queue instead of merging directly; the queue's configured merge method wins and `--merge-method`/`--squash`/`--rebase` are ignored with a warning. Queued PRs merge as the queue processes them, which can land across more than one merge group.
- **Auto-merge every open PR in the stack** so the queue drains without polling for a human to click merge: `gh pr merge <pr-number> --auto --squash` (or the org's standard method) on every open PR in the stack, not just the bottom one. Re-assert after every `gh stack submit` — a force-push clears a PR's auto-merge flag.
- **A PR ejected from the merge queue ejects every PR above it in the stack too.** Re-add the stack to the queue once the underlying issue (failing required check, merge conflict) is resolved.
- **Programmatic/API merges must use the asynchronous merge endpoint.** Legacy synchronous REST/GraphQL merge calls cannot merge a stack; only `gh stack merge`, `gh pr merge`, the website, or the async merge API work. An API stack merge is atomic (the requested range merges or queues together, or none of it does) and runs in the background — poll for the result rather than expecting synchronous completion.
- **Keep watching through every merge.** Green now does not mean green later: as lower PRs merge, the branches above rebase onto the new trunk and can pick up new conflicts or CI failures. Re-check `mergeable` / `mergeStateStatus` on every remaining PR after each merge; never assume a once-green upper PR stays mergeable through the rest of the drain.

## Hard rules

1. **Use `gh stack`, not raw `git`/`gh pr create`, for anything that creates or moves stack branches or opens/updates their PRs.** Raw `git checkout -b`, a `git push` that opens a new branch, and `gh pr create` all produce a PR with no stack relationship. Plain `git add`/`git status`/`git log`/`git diff` for inspection, and plain `git commit` on an already-tracked stack branch, are fine.
2. **Plan the stack before writing code.** An unwritten plan is no plan. Build bottom-up against the plan with `gh stack init` / `gh stack add`.
3. **One logical change per branch; each branch leaves the build green and ships its own tests.**
4. **A commit on a lower branch does not auto-propagate.** After changing a lower layer, explicitly cascade with `gh stack rebase --upstack` (or `gh stack sync`) and push with `gh stack push` before assuming the branches above it are current.
5. **Submit and monitor the whole stack** (`gh stack submit --auto`, `gh stack view`, `gh pr view` per branch), not one PR in isolation.
6. **Never force-push raw** (`git push --force`). `gh stack push` / `gh stack submit` already force-push with lease; do not bypass them.
7. **Prefer `gh stack rebase` over the website's "Rebase stack" button from automation** whenever the repository requires signed commits — server-side rebases produce unsigned commits.
8. **Auto-merge every open PR in the stack**, not just the bottom one, and re-assert it after every `gh stack submit` (a force-push clears the flag).
9. **Merges go through `gh stack merge` / `gh pr merge` / the website / the async merge API.** The legacy synchronous merge API cannot merge a stack.
10. **Cross-fork stacks are not supported.** Every branch in a stack must live in the same repository.

## Reference: `gh stack` exit codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Generic error |
| 2 | Not in a stack, or stack not found |
| 3 | Rebase conflict |
| 4 | GitHub API failure |
| 5 | Invalid arguments or flags |
| 6 | Disambiguation required (branch belongs to multiple stacks) |
| 7 | Rebase already in progress |
| 8 | Stack is locked by another process |
| 9 | Stacked pull requests are not enabled for this repository |
| 10 | Modify session interrupted; recovery required |
