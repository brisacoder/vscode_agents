---
name: github-stacking
description: "Native gh-stack command expert: cached insert/move/swap/fold, publication, recovery, feedback/CI, and merge recipes. No trial-and-error discovery."
user-invocable: false
context: inline
---

# GitHub Stacking — Native Command Expert

**Choose the intent below and follow its recipe. Do not conduct command archaeology, guess flags, or try a sequence of plausible subcommands.**
Use GitHub's own `github/gh-stack` extension, not similarly named third-party stack tools.
Load `stack-shepherding` for publication/review/CI/completion obligations. Use the caller's stack plan; decomposition and quality budgets belong to PR Stack Planner.

Load `implementation-first` before coding/delegation. **DO NOT create/run tests during implementation; hand the complete agreed code to the user before the approved test phase.**
Test/benchmark/coverage mandates in bundled recipes are approved-phase ONLY. Early ready PR publication and truthful current-CI observation still apply.

## Intent → recipe

| You want to… | Exact entrypoint / next step |
|---|---|
| Start + first ready PR | `gh stack init --base <trunk> <branch>`; commit; immediately `gh stack submit --auto --open`; [recipe](references/cookbook.md#publish-the-first-layer). |
| Add a top layer | `gh stack top`, then `gh stack add <branch>`; commit; **immediately** `gh stack submit --auto --open`; [recipe](references/cookbook.md#append-a-top-layer). |
| Insert in the middle | `gh stack modify`; select the neighbor; `i` below / `I` above; [insert recipe](references/cookbook.md#insert-a-middle-layer). |
| Move a layer | `gh stack modify`; select it; `Shift+Up` / `Shift+Down` (also `K` / `J`); [move recipe](references/cookbook.md#move-or-swap-layers). |
| Swap two layers | In the same reorder-only editor, perform the exact moves in the [swap recipe](references/cookbook.md#move-or-swap-layers); no standalone swap command. |
| Fold two layers together | `gh stack modify`; select the source; `d` into lower / `u` into higher; [fold recipe](references/cookbook.md#fold-drop-or-rename). |
| Drop or rename a layer | `gh stack modify`; `x` / `r`; [recipe and preserved-PR warning](references/cookbook.md#fold-drop-or-rename). |
| Repair a branch-only row / draft PR | `gh stack submit --auto --open`, then the [publication gate](references/cookbook.md#verify-publication). |
| Adopt/link existing branches or PRs | `gh stack init <branches-bottom-to-top>` or `gh stack link --open <targets-bottom-to-top>`; [recipe](references/cookbook.md#adopt-existing-work). |
| Fix a lower branch and cascade | `gh stack checkout <owner>`, commit, `gh stack rebase --upstack`, `gh stack submit --auto --open`; [recipe](references/cookbook.md#update-a-lower-layer). |
| Continue/abort a paused operation | `gh stack rebase --continue/--abort` or `gh stack modify --continue/--abort`; [recovery](references/cookbook.md#recover-without-guessing). |
| Refresh after a lower merge | `gh stack sync`, inspect survivors, verify current heads; [recovery](references/cookbook.md#recover-without-guessing). |
| Reply AND resolve a review conversation | Exact paginated queries, reply, resolution mutation, and readback in [feedback recipes](references/feedback-ci.md#reply-resolve-and-read-back). |
| Act on failures while CI is running | Inspect individual jobs/steps, annotations and statuses via [CI recipes](references/feedback-ci.md#inspect-distributed-ci-now); do not wait for the whole run. |
| Merge/queue an authorized ready range | `gh stack merge <PR-or-stack-number> --yes --squash`; [merge recipe](references/cookbook.md#merge-only-an-authorized-ready-range). |
| See every command/flag/exit code | [Complete command reference](references/commands.md); load only the relevant section. |

## One capability check, not an expedition

Recipes target **gh-stack v0.2.1**, researched against official documentation, its release source, and actual CLI simulations.
At session startup check `gh stack --version`, `git --version`, authentication, and `gh stack view --json` once; reuse the result until capabilities change.
The pinned extension reference requires `gh` 2.0+ and Git 2.36+; `gh skill install` is optional and requires a CLI that actually supplies that command.

For an older/different version, inspect **only** `gh stack <relevant-subcommand> --help` and the linked official CLI reference once.
If the recipe is unsupported or the documented recovery fails, report the exact capability/error and request an authorized upgrade or interactive handoff.
Do not search commit history, issues, source trees, alternative stack tools, or invent flags to rediscover a documented operation.
This discovery limit does not cap legitimate conflict-resolution steps or the policy's bounded fix/retry loop.

## Non-negotiable mechanics

- **Every agent submission is `gh stack submit --auto --open`.** Bare `--auto` creates drafts; neither `push` nor `sync` creates missing PRs.
- **A local branch is not a published stack entry.** Verify its PR number/URL, non-draft state, head/base, and remote membership; a successful exit can contain failed-PR warnings.
- `modify` is interactive. There are no standalone insert/move/swap/fold subcommands, no documented headless plan flag, and no submit dry-run flag.
- Have a clean affected working tree, linear history, no paused Git operation, and no queued PR before restructuring. Do not auto-stash other people's work.
- Save TUI changes with `Ctrl+S`; cancellation/undo is not publication. Inspect the result, then submit with `--auto --open` and verify it.
- A lower commit does not automatically cascade. Rebase descendants explicitly before publishing updates.
- Native stacks currently **do not support auto-merge or rule bypass**. Do not loop on `gh pr merge --auto` or use legacy synchronous merge APIs.
- Pushes/submission are lease-protected but **not globally atomic**. On partial failure, inspect actual branch/PR state before retrying.
- All layers must be in the same repository; cross-fork stacks and cross-stack moves are not supported.
- Never edit catalog/recovery journals or raw-force-push to manufacture a stack. Preserve identities and use the matching documented recovery command.

## Authoritative references and test boundary

- [GitHub PR how-tos](https://docs.github.com/en/pull-requests/how-tos) → [Stacked PRs](https://docs.github.com/en/pull-requests/how-tos/stacked-pull-requests).
- [CLI reference](https://docs.github.com/en/pull-requests/reference/stacked-prs-cli-commands).
- [Create](https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/creating-stacked-pull-requests),
	[manage](https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/managing-stacked-pull-requests),
	[review](https://docs.github.com/en/pull-requests/how-tos/review-pull-requests/reviewing-stacked-pull-requests),
	[merge](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/merging-stacked-pull-requests),
	[troubleshoot](https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/troubleshooting-stacked-pull-requests).
- [Release v0.2.1](https://github.com/github/gh-stack/releases/tag/v0.2.1) and [simulation evidence](references/validation.md).

The upstream quickstart and CLI reference currently state different version floors; use the pinned extension reference and installed capabilities, not speculative upgrades.
High-level tutorial examples may defer publication or mention auto-merge; this workspace's early-review policy and pinned operational merge reference take precedence.
Tests prove local CLI/Git behavior with a fixture API, not live GitHub permissions, branch protection, Copilot execution, or queue success. Never claim an unperformed live test.
