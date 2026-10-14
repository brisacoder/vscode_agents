# Native CLI Recipe Verification

## Verified target

- Official release: `github/gh-stack` **v0.2.1**.
- Release source: `29a36d314be664583705f18e41cebdf2b61ad514`.
- Frontend/runtime: GitHub CLI 2.45.0, Git 2.43.0, Linux.
- Every supported root/subcommand help page was read from the actual v0.2.1 binary and checked against official CLI/how-to documentation.

## Executed simulations

**18 pytest cases passed: 13 real stack CLI cases and 5 document/API recipe checks.**
Commands ran through the actual `gh stack`/`gh api` frontend and official extension binary, not a replacement implementation of the CLI.
Local Git refs, commits, pushes, rebases, catalogs, and modify recovery state were real.
Modify was driven through a pseudo-terminal with actual keys and rendered-screen checks.

| Case | Observed result |
|---|---|
| First commit → immediate submit `--auto --open` on each of three layers | Each layer acquired a ready PR immediately; bases were `main`, lower layer, lower layer; CLI view carried PR identities. |
| Bare `submit --auto`, then `--auto --open` | First created drafts; second converted existing drafts to ready. |
| Insert below `i`, commit/cascade/submit | Middle order and bases correct; empty inserted row initially had no PR, then acquired one after content/publication. |
| Insert above `I`, commit/cascade/submit | Equivalent adjacent insertion direction verified with distinct expected order. |
| Move up via Shift+Up | Selected layer swapped toward top; resulting refs/PR bases correct. |
| Move down via Shift+Down | Selected layer swapped toward trunk; resulting refs/PR bases correct. |
| Non-adjacent swap via staged adjacent moves | `a → b → c` became `c → b → a`; top tree retained all independent fixture content. |
| Fold down `d` | Source left membership; lower receiver retained both layers and descendants remained linear. |
| Fold up `u` | Higher receiver retained both layers; remaining membership/bases correct. |
| Submit an empty added branch | CLI returned 0 with a failed-creation warning; missing PR remained detectable. Exit success is not a publication gate. |
| Push/sync unsubmitted committed layers, then submit | Push/sync created no PRs; submit created the PRs and native fixture stack. |
| Link/navigation/local unstack | Ready PRs created without local tracking edits; navigation/path correct; local unstack preserved remote state and refs. |
| `add -m` on a nonempty top | New content committed on the new branch, not its parent; ready publication and PR association verified. |

Additional checks validated YAML frontmatter, bundled links/anchors, all 20 real command help entrypoints, and known consumer contradictions.
Real `gh api` fixture calls verified that a reply alone does not resolve a thread, resolution/readback do, and denied resolution remains an error/pending obligation.
The current-attempt jobs recipe exposed **10 failures while 20 sibling jobs and the workflow were still running**.
The Copilot REST recipe encoded the documented bot reviewer as the required array. These checks test protocol syntax/available evidence, not an LLM policy decision.

## Isolation and limits

Git SSH operations were redirected to a disposable local bare repository. API traffic used an isolated Unix-socket fixture server and a dummy non-secret authentication profile.
The server implemented only the upstream routes needed by these cases; it had no forwarding path to GitHub. No production branches, PRs, review threads, or queues were changed.

These tests verify client/local-history behavior and recipe sequencing. Synthetic API responses **do not verify** live permissions, branch protection, Copilot execution, real queue admission,
provider workflow triggering, or the exact live server error for an empty diff. Those postconditions must be checked in the consuming repository.
Merge, remote unstack, alias installation, and upstream feedback submission were documented/help-checked, not executed against live resources.

## Maintaining the recipe cache

When changing the supported release, rerun the exact affected recipes in a disposable fixture and update the matching reference only after inspecting actual state.
Normal consuming-agent runs should use the cached intent recipe and one capability check, **not repeat this research or recreate these simulations**.
Keep official links as targeted escalation references; do not chase commit history to decide a documented command's syntax.