---
name: stack-shepherding
description: The mandatory, non-negotiable contract for shepherding a submitted Graphite stack all the way to merged. Load at startup in every PR-lifecycle agent (PR Review Resolver, PR Watch Agent, PR Stack Planner) and obey it as the single source of truth for the six shepherding rules that the agents' longer bodies must never contradict. It exists because these rules were previously scattered across hundreds of lines in three agents, where the load-bearing ones got buried and ignored: the stack was submitted, went green, and the agent stopped -- leaving unaddressed Copilot/LLM/human review comments, unresolved threads, no auto-merge set, and open PRs that never merged. The six rules: (1) set auto-merge / "merge when ready" on EVERY PR the moment a stack is created or submitted, always; (2) address ALL review comments across the WHOLE stack by default -- Copilot, other LLM reviewers, and humans -- without being asked; (3) review is iterative, not one-shot -- a fix triggers a fresh review that adds new comments, so loop until a poll yields zero unaddressed comments AND zero open PRs; (4) when a comment is addressed by a code change and/or a reply, RESOLVE its conversation; (5) every PR carries a Jira ticket key in plain ASCII (e.g. OCTO-1234); (6) submitted-and-green is NOT done -- merged is done, and conflicts/rebases/merges keep happening, so babysit the stack continuously until every PR is merged or closed. This skill owns the WHAT and the WHEN; the graphite-stacking skill owns the gt/gh HOW.
user-invocable: false
context: fork
---

# Stack Shepherding — Drive the Stack to Merged

This is the binding contract for any agent that owns a pull-request **stack** after it is submitted. It is deliberately short. If a longer instruction elsewhere in your body contradicts a rule here, **this skill wins** — these are the rules that kept getting buried and ignored.

The one sentence that governs everything: **a submitted, green stack is the *middle* of the job, not the end.** The job ends only when every PR in the stack is `merged` or `closed`. Until then you are actively shepherding it — setting auto-merge, addressing every review comment, resolving threads, triggering required workflows, and rolling the stack forward through every merge.

The `graphite-stacking` skill owns the `gt`/`gh` mechanics (the *how*). This skill owns the *what* and the *when*. Load both.

## The Six Rules

### 1. Always set auto-merge on every PR. Always.

The moment a stack is created or (re-)submitted, enable auto-merge ("merge when ready") on **every** open PR in the stack — not just the bottom one, not "once it's ready", not "when the user asks". It is unconditional.

```sh
# every open PR, every submit (idempotent)
for pr in <all-open-pr-numbers>; do gh pr merge "$pr" --auto --squash; done
```

GitHub's native merge queue holds each PR until its base merges and its required checks pass, then admits them in order. Setting auto-merge on the whole stack up front is what makes it drain without you babysitting each individual merge. A force-push (any `gt submit --stack` cascade) can clear a pending auto-merge request, so **re-assert it after every submit**. An open PR without auto-merge set is a defect you fix this iteration.

### 2. Address every review comment, across the whole stack, by default.

You do **not** wait to be told to check comments. On every poll, enumerate **all** review comments on **every** PR in the stack — line-level review comments, PR-level issue comments, and review-summary bodies — from **every** author: the Copilot reviewer, other LLM/bot reviewers, and humans. `github-copilot[bot]` and other bots are treated exactly like human reviewers; author identity never downgrades a comment.

The unit of work is **every unresolved comment in the stack**, found by enumeration — not "comments newer than when I started". Do not baseline-and-ignore pre-existing comments: a comment that was already on the PR when you began is still your responsibility. Reconcile the full comment set against your ledger every poll; any comment without a terminal disposition is open work.

Each comment is driven to one terminal disposition and gets its **own threaded reply** (never a single aggregated roll-up comment):
- **fixed** — a code change addressed it; the reply states what changed and links the fix commit SHA.
- **already-correct** — the concern does not apply; the reply cites the evidence.
- **wont-fix** / **deferred** — the reply gives the rationale (and the tracked follow-up issue, if deferred).
- **awaiting-user** — needs a human decision; the reply states exactly what decision and tags the user, so the thread is never silent.

A comment left with no reply is the defect this rule exists to prevent.

### 3. Review is iterative, not one-shot.

Fixing a batch of comments pushes new commits, which triggers the reviewer (Copilot especially) to **review again and leave new comments**. You are not done when you have addressed the first round. After every fix-and-push, expect a fresh review and keep polling: address the new comments, push again, expect another review. The loop terminates only when a poll finds **zero unaddressed comments AND zero open PRs** — not when the first batch is cleared.

### 4. Resolve the conversation once a comment is addressed.

When a comment has been addressed — a code change landed and/or a reply was posted giving its terminal disposition — **resolve that review thread**. Use `resolveReviewThread` (or the GraphQL `resolveReviewThread` mutation via `gh api graphql`). Resolving is the visible signal to the reviewer and to the merge gate that the thread is handled; leaving addressed threads unresolved makes the PR look like it still has open feedback and is a common reason a stack looks "stuck". Resolve `fixed`, `already-correct`, `wont-fix`, and `deferred` threads (each after its reply is posted); leave `awaiting-user` threads **open** (they genuinely await a human).

### 5. Every PR carries a Jira ticket key in plain ASCII.

Every PR title or body must contain a Jira key in **plain ASCII** matching `[A-Z][A-Z0-9]+-[0-9]+` (e.g. `OCTO-1234`) — not a smart-quoted, zero-width, or otherwise non-ASCII variant. The org-required `Require Jira Ticket` workflow blocks the merge queue without it, so a PR with no valid key can never merge. Never invent a ticket number: if none is known, surface it as awaiting-user and stop on that PR until a human supplies the key, then set it with `gh pr edit`.

### 6. Submitted + green is not done. Merged is done.

The single most expensive mistake these agents make is concluding "the stack is submitted and green, so we're finished." You are not. Green is a snapshot, and it does not hold: the merge queue admits PRs one at a time, every merge rebases the survivors, and a rebase can re-introduce conflicts, re-fire CI, or strand a PR on a missing required check. A PR that is green at PR 1 can need work by PR 20.

So you babysit the stack **continuously** until every PR is `merged` or `closed`:
- After each merge, roll the stack forward (`gt sync` + `gt submit --stack`), re-assert auto-merge (Rule 1), and keep watching.
- If a required org workflow (e.g. `Require Jira Ticket`, `Trivy Scan`) is **absent** for a PR's head SHA, the queue will never admit it — trigger it autonomously (`gh workflow run` / `gh run rerun`, else a no-op amend `gt modify -c -m "chore: re-trigger required workflows"` + `gt submit --stack`). Never idle for a human to notice. See the `graphite-stacking` skill, *Drain the stack* and *Restack cascades and stuck PRs*.
- An `APPROVED` / `CLEAN` / `QUEUED` PR is **done being reviewed** — the review requirements, including any "1 review from a code owner" rule, are already satisfied. Do not investigate `CODEOWNERS` or chase which owner must approve; set auto-merge and let the queue take it.

There is no idle state in which an open PR is making no progress and the agent is content. Either a PR is progressing toward merge (queued, CI running, or rebasing), or you are doing something this iteration to make it progress, or it is genuinely blocked on a human and surfaced as such.

## Termination

You are done — and only done — when **every PR in the stack is `merged` or `closed`**, AND every review comment across the stack has a terminal disposition with its reply posted and its thread resolved (except `awaiting-user`, which stays open with a posted question). Green CI, a clean restack, and "mergeable" are necessary but **not sufficient**; the merge itself is the terminal event. If progress genuinely stalls only on items outside the agents' control (a human decision, a required human review), surface them explicitly and keep monitoring — never silently exit.

## What this skill does not own

- The `gt`/`gh` command syntax, the merge-queue admission details, restack-cascade quiescing, and the missing-check recovery ladder — those are the `graphite-stacking` skill.
- The six **PR-shape** rules (2,000-line cap, plan-first, formatters, freshness, 75% coverage, 300-line files) — those are the PR Stack Planner's own rules. This skill is about shepherding a stack *to merged*, not about its size or shape.
