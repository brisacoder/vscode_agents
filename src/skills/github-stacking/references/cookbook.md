# Native Stack Cookbook — Choose the Intent

Target: official `github/gh-stack` v0.2.1. Use the command reference for flags, not source/issue searches.
Names below are placeholders: substitute the caller's actual repository, trunk, branches, PRs, and authorized commit subjects.
Stack order in prose is **bottom → top**; the modify editor displays **top → bottom**.

## Publish the first layer

1. From the chosen trunk, run `gh stack init --base <trunk> <first-branch>`.
2. Stage only intended paths with `git add -- <paths>` and create the first content-bearing `git commit -m "<subject>"`.
3. **Before the next task**, run `gh stack submit --auto --open`.
4. Run the publication gate below and request Copilot review using the feedback recipe.

One layer already needs its ready PR. GitHub creates a remote stack object only when at least two PRs exist; do not delay the first PR to obtain that object.

## Append a top layer

1. `gh stack top`.
2. `gh stack add <next-branch>`; it must run from the current top, not an arbitrary middle layer.
3. Commit the next layer's actual changes, then **immediately** `gh stack submit --auto --open`.
4. Verify every active committed branch's PR and the native remote order, not only the newest PR.

Prefer explicit add + commit over `add -Am`: on an empty current layer the shortcut commits there; on a nonempty one it creates a new layer and commits there.
`-A` also stages untracked paths; never use it to sweep unrelated work into a branch.

## Insert a middle layer

Example: existing `main → a → b → c`; insert `new` between `a` and `b`.

1. Preflight: clean affected worktrees, linear history, no paused Git operation, and no queued PR. Record current order/PR identities.
2. `gh stack checkout b`, then `gh stack modify`.
3. Cursor starts on the checked-out branch. Select `b` if needed with arrows or `j`/`k`; press **`i`** (insert below, toward trunk).
4. Type `new`, press **Enter**, inspect the preview `main → a → new → b → c`, then **Ctrl+S**.
   Equivalent: select `a` and press **`I`** (insert above, away from trunk). Do not use `gh stack add` for a middle insertion.
5. `gh stack view --json` verifies the local order. The inserted branch is initially empty and has **no PR**; this is not publication success.
6. `gh stack checkout new`; commit the intended content. Then `gh stack rebase --upstack` and **immediately** `gh stack submit --auto --open`.
7. Verify all PRs, updated bases, remote membership/order, and cleared pending-modify state. Never stop with `new` shown only as a branch row.

Empty commits do not manufacture a meaningful PR diff. If no content exists yet, keep the entry unstarted in the plan.

## Move or swap layers

Use `gh stack modify`, select a branch, and move it one adjacent position at a time:

- **Shift+Up** or **`K`**: away from trunk, toward the top.
- **Shift+Down** or **`J`**: toward trunk, toward the bottom.

### Move one entry

Example `main → a → b → c`: select `b`, press Shift+Up once, inspect `main → a → c → b`, Ctrl+S, then `gh stack submit --auto --open`.
Moving it toward trunk instead gives `main → b → a → c`. Reordering rebases affected histories; validate actual dependencies and tests afterward.

### Swap adjacent entries

Select the lower of the two; one Shift+Up swaps them. Inspect, Ctrl+S, submit `--auto --open`, and verify PR bases/membership.

### Swap non-adjacent entries without reversing the middle

Example `main → a → b → c` → `main → c → b → a`:

1. Select `a`; press Shift+Up twice. Preview becomes `main → b → c → a`.
2. Select `c` (one `j` from selected `a` in this top-to-bottom editor); press Shift+Down once. Preview becomes `main → c → b → a`.
3. Ctrl+S, `gh stack submit --auto --open`, verify all bases/order and the top tree's retained content.

General rule: move the lower endpoint to the upper endpoint's position; then move the original upper endpoint to the original lower position.
There is **no standalone move or swap subcommand**. Reordering and structural edits cannot share one session: save/submit one, then start another.

## Fold, drop, or rename

| Intent | Actual editor recipe | What to verify |
|---|---|---|
| Fold `b` into lower `a` | `gh stack checkout b`; `gh stack modify`; select `b`, **`d`**, Ctrl+S. | `a` receives both layers; `c` is rebased; membership becomes `a → c`. |
| Fold `b` into higher `c` | Select `b`, **`u`**, Ctrl+S. | `c` retains both layers; membership becomes `a → c`. |
| Drop `b` and its unique changes | Select `b`, **`x`**, Ctrl+S. | Descendants exclude `b`'s unique commits; `b`'s branch/PR are preserved outside the resulting stack. |
| Rename `b` | Select `b`, **`r`**; replace the inline name, Enter, Ctrl+S. | New local name, expected commit content, resulting PR identity and branch bases after submission. |
| Undo a staged action | **`z`**, inspect preview. | No intended change was lost from the remaining plan. |
| Cancel before applying | **`q`** or Esc. | No structural edit applied. |

After applying any edit: `gh stack submit --auto --open`, then publication verification and impacted tests.
Fold is not “squash every commit in a branch”: it combines layers. Dropped/folded refs and PRs are preserved, not automatically closed/deleted.
Reconcile displaced PRs explicitly and retain their feedback obligations; close only under the caller's authority, with a replacement explanation where appropriate.
Renaming a submitted head can require a replacement PR; never promise preservation of its PR number. Reconcile old/new PRs and review conversations.
Do not drop required dependencies, fold into trunk, or rewrite merged/queued entries. The editor cannot split a branch or move it to another stack.

## Verify publication

Run `gh stack view --json`. In v0.2.1, `.branches` is bottom-to-top; rows contain `.name`, `.pr.number`, `.pr.url`, `.pr.state`, and `.needsRebase`.
It does **not** contain `isDraft`; verify that remotely for every committed active row:

- `gh pr view <pr> --repo <owner>/<repo> --json number,url,state,isDraft,headRefName,baseRefName,headRefOid`.
- Require the intended open PR, nonempty number/URL, `isDraft: false`, intended head, and the correct lower-branch/trunk base.
- Compare the remote `headRefOid` to the actual local branch tip, not `.branches[].head` (which may be absent).
- With two or more PRs, `gh api repos/<owner>/<repo>/pulls/<pr> --jq .stack` verifies native membership/position.
  Query the reported `gh api repos/<owner>/<repo>/stacks/<stack-number>` and compare ordered PR numbers/head branches to the intended local order.

Missing PR after a content-bearing commit: inspect submission warnings and base→head diff, fix the specific cause, rerun `gh stack submit --auto --open`, then recheck.
If it is merely draft, `gh pr ready <pr> --repo <owner>/<repo>` is the targeted correction; normal stack submission still uses `--auto --open`.
If existing PRs are not natively linked, use the adoption/link recipe; do not declare a plain branch chain a native stack.
Neither `gh stack push` nor `gh stack sync` repairs missing PR creation. A zero CLI exit can still carry PR/stack warnings; the postconditions decide success.

## Adopt existing work

- Local tracking: `gh stack init --base <trunk> <a> <b> <c>` adopts existing names automatically, bottom-to-top; then `gh stack submit --auto --open` and verify.
- Remote-only linking: `gh stack link --open <PR-a> <PR-b> <PR-c>` or branch names in that order.
  This can push branch arguments, open PRs, and correct bases, but **does not create local tracking**; use explicit `gh stack checkout <PR-or-stack>` to import it afterward.
- Append to a known remote stack: `gh stack link --open <stack-number> <new-PR-or-branch>`. This is additive; it is **not** a middle-insert/reorder recipe.
- Import a remote stack: `gh stack checkout <stack-number>` or an unambiguous PR URL/number.

Do not use a guessed `--adopt` flag. Do not initialize pre-created empty planned branches and report them as published PRs.

## Update a lower layer

1. `gh stack checkout <owning-branch>`.
2. Stage intended files and commit the fix normally.
3. `gh stack rebase --upstack` (or `--no-trunk --upstack` only when intentionally excluding trunk/network updates).
4. `gh stack submit --auto --open`; verify published head SHAs, membership, and impacted tests/CI.
5. Return the published fix SHA/explanation to the lifecycle controller for its reply-and-resolve step.

A plain `git commit` does not cascade. `sync` cascades when trunk moves; it is not a substitute for propagating a newly edited lower branch explicitly.

## Recover without guessing

| Observed condition | Documented response |
|---|---|
| Rebase conflict | Resolve/stage in the reported owner worktree; `gh stack rebase --continue`; repeat for actual subsequent conflicts, or `--abort`. |
| Modify conflict/interruption | Resolve/stage in the reported owner; `gh stack modify --continue`, or `--abort` to restore touched state. |
| Modify already applied, pending submit | Finish `gh stack submit --auto --open`; do not delete the journal or expect `modify --abort` to undo a completed pending-submit edit. |
| Lower PR merged / remote ahead | `gh stack sync`, verify survivors; use `--prune` only when authorized to delete merged local branches. |
| Sync conflict | Sync restores its cascade; use `gh stack rebase` to resolve deliberately, then submit `--auto --open` and verify. |
| Local/remote compositions diverged | Non-interactive sync may exit 0 without updating anything. Inspect state; choose the documented source-of-truth/unstack recovery only with authority. |
| Lease rejection / partial submit | Inspect actual remote refs/PRs, reconcile competing edits, retry the documented command; do not raw-force-push. |
| Dirty/busy/foreign worktree | Stop the mutation and identify its owner/path; do not auto-stash, steal a checkout, or erase changes. |
| Target checked out elsewhere | Supported navigation `--print-path` returns its owning path; check exit status, quote the path, then navigate there. |
| Catalog/journal lifecycle mismatch | Use the matching build in the recorded owner to continue/abort. Never edit/delete the journal or mix old/new builds in one clone. |
| Auth/permission/feature failure | Report the precise error and needed authorization/capability; no anonymous fallbacks or speculative alternate flags. |

Unstacking is not a casual recovery shortcut: it changes native membership. Preserve PRs/refs, record intended order, and obtain authority for the exact change.
`gh stack unstack --local` affects only local tracking and does not reconcile the remote composition.

## Merge only an authorized ready range

1. Verify implementation completion, required reviews/checks, linear ancestry, and addressed/resolved conversations on the actual current heads.
2. Select the exact bottom-up range using `gh stack merge <top-PR-in-range> --yes --squash` (or the repository's supported merge method).
   `gh stack merge <stack-number> --yes` selects the whole stack; it is not a per-PR operation on an arbitrary middle layer.
3. If the repository uses a merge queue, GitHub queues the range and chooses the merge method; supplied method flags may be ignored.
4. Observe actual merged/queued/failure results. After lower merges, sync and re-evaluate survivors; queued is not merged and a removed lower PR can eject descendants.

No rule bypass and no auto-merge for native stacks. `gh pr merge --auto` and legacy synchronous REST/GraphQL merge calls are not substitutes.
Do not merge unfinished early-review PRs or assume an entire queued range lands atomically; inspect actual outcomes after failures.