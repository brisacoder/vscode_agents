# Complete gh-stack Command Reference

Verified against official `github/gh-stack` **v0.2.1** help and the [GitHub CLI reference](https://docs.github.com/en/pull-requests/reference/stacked-prs-cli-commands).
This is syntax/capability reference, not permission to mutate. `<...>` values must be substituted; `[ ... ]` denotes optional syntax, not literal brackets.
Every command supports `-h` / `--help`; the root supports `-v` / `--version`. No invented global `--yes`, `--json`, `--dry-run`, or headless editor flag.

## Setup

- `gh extension install github/gh-stack` installs the official extension; `gh extension upgrade stack` upgrades only when authorized.
- `gh auth status --hostname <host>` checks existing authentication; login/secret entry belongs to the user, never a hidden automated fallback.
- `gh skill install github/gh-stack` is optional on a CLI that provides `gh skill`; it does not install this repository's policy bundle.
- The v0.2.1 README/CLI reference states `gh` 2.0+, Git 2.36+. High-level quickstart version floors differ; verify the installed capabilities once.

## Stack management

| Command / arguments | Supported options | Exact effect / caveat |
|---|---|---|
| `gh stack init [branches...]` | `-b`, `--base <trunk>` | Adopt/create bottom-to-top; explicit names avoid prompts; enables rerere. |
| `gh stack add [branch]` | `-A`/`--all`, `-u`/`--update`, `-m`/`--message <subject>` | Append at current top; stage options are mutually exclusive. |
| `gh stack view` | `-s`/`--short`, `--json` | Default view may be interactive; short prints branch+PR context, JSON is machine-readable and avoids picker prompts. |
| `gh stack checkout [target]` | `--print-path` with explicit target | Stack/PR number, PR URL, or branch; import remote tracking if needed. |
| `gh stack modify` | `--continue`, `--abort` | Interactive insert/drop/fold/reorder/rename; recovery flags are not headless edit arguments. |
| `gh stack unstack [stack-number]` | `--local`; alias **`delete`** | Authorized membership removal, not PR/ref deletion; locked members may remain. |

`add -m` without a name generates a date+slug name. On an empty current layer it commits there; otherwise it creates a new layer for that commit.
Always supply `-m` with `-A`/`-u` in agent recipes. Actual v0.2.1 help permits an editor when `-m` is omitted; do not mistake the no-prompt recipe for an unsupported-flag claim.
Checkout resolves bare numbers as stack/PR identifiers before branch names; use an explicit PR URL for unambiguous remote import.
Adoption of a branch already owned by another worktree does not steal its checkout; stage/commit shortcuts cannot operate on that foreign-owned layer.

## Modify editor — complete operations

| Key | Operation |
|---|---|
| Up/Down or `k`/`j` | Select a displayed row; screen order is top-to-bottom. |
| `i` / `I` | Insert below toward trunk / above toward top; type name and Enter. |
| Shift+Up / Shift+Down (also `K` / `J`) | Move selected row one adjacent position away from / toward trunk; repeat to move farther or swap endpoints. |
| `d` / `u` | Fold selected layer into lower / higher neighbor. |
| `x` | Drop selected layer's membership/unique commits from the resulting stack. |
| `r` | Rename through inline prompt; Enter stages the new name. |
| `z` | Undo last staged action. |
| `c` / `f` | Expand commits / changed files. |
| `?` | Editor help overlay. |
| Ctrl+S | Apply staged changes; inspect actual resulting order and submit `--auto --open`. |
| `q` / Esc | Cancel before apply. |

Structural edits (insert/drop/fold/rename) and reorder operations are mutually exclusive within one editor session.
Clean affected worktrees, linear history, an active local stack, no paused Git operation, and no queued PR are prerequisites.
There are **no standalone insert, move, swap, fold, rename, split, or cross-stack move commands**. The editor cannot split a branch.
Without interactive-terminal capability, use an interactive handoff for restructuring; do not type pretend flags or hand-edit `.git/gh-stack`.

## Remote operations

| Command / arguments | Supported options | Exact effect / caveat |
|---|---|---|
| `gh stack submit` | `--auto`, `--open`, `--remote <name>` | Push/create/update/link; use `--auto --open` together and verify publication. |
| `gh stack sync` | `--remote <name>`, `--prune` | Fetch/reconcile, fast-forward trunk, cascade when trunk moved, push and sync existing PR/stack state; never create a missing PR. |
| `gh stack rebase [branch]` | `--downstack`, `--upstack`, `--no-trunk`, `--continue`, `--abort`, `--remote <name>` | Cascade/recover; `--no-trunk` skips fetch/trunk. |
| `gh stack rebase [branch]` | `--committer-date-is-author-date`, alias `--preserve-dates` | Preserve committer dates through the merge backend; continuation uses saved options. |
| `gh stack push` | `--remote <name>` | Push active non-merged/non-queued branches with per-branch leases. Not atomic and does not create/update PRs. |
| `gh stack link <target> <target>...` | `--base <trunk>`, `--open`, `--remote <name>` | Additive remote linking/appending; no local tracking/reorder. |
| `gh stack merge [stack-or-PR-number]` | `--merge-method <method>`, `--merge`, `--squash`, `--rebase`, `-y`/`--yes` | Authorized bottom-up merge/queue; no bypass. |

`submit --auto` alone creates drafts. Interactive submit normally creates ready PRs but allows per-row draft/deselection; use the explicit noninteractive recipe to avoid omissions.
Link targets are branch names, PR numbers/URLs, or an existing stack number first; `--base` is ignored when appending to an existing stack.
Merge methods are `merge`, `squash`, or `rebase`; a configured queue chooses the method and may ignore supplied flags. Native auto-merge is unsupported.
Native stack membership needs at least two PRs; a first single-layer ready PR is valid publication before that object exists.
Fully merged stacks cannot be extended; submission forks remaining new work into a new stack on trunk.
`submit` can emit warnings and exit 0 with incomplete publication; `sync` divergence can exit 0 without updates. Check results, not only exit codes.
Normal push/submit may update earlier refs/PRs before a later failure; do not assume global rollback or overwrite another author's lease-rejected work.

## Navigation

| Command | Options / behavior |
|---|---|
| `gh stack up [n]` | Default 1; away from trunk; `--print-path` supported. |
| `gh stack down [n]` | Default 1; toward trunk; `--print-path` supported. |
| `gh stack top` | Furthest active layer; `--print-path` supported. |
| `gh stack bottom` | Closest active layer; `--print-path` supported. |
| `gh stack trunk` | Chosen trunk; invoke from an unambiguous stack branch; `--print-path` supported. |
| `gh stack switch` | Interactive picker; no numeric move/reorder effect and no headless flag. |

Navigation skips merged layers and clamps at bounds. `--print-path` emits the raw absolute path only on success; check status and quote it before changing directories.
An occupied foreign target prints its path with that flag without switching either worktree; an unoccupied target may be checked out locally before printing.
Without the flag, a foreign-owned target fails with its owner path. Never steal checkouts or mix gh-stack catalog versions in one clone.

## Utilities

| Command | Options / effect |
|---|---|
| `gh stack alias [name]` | Default name `gs`; `--remove` removes the chosen alias; installs a user-local forwarding wrapper, so obtain authority. |
| `gh stack feedback [title]` | Opens upstream feedback/discussion; not a command-discovery shortcut or an automatic support submission. |

Theme variables: `GH_STACK_THEME=auto/light/dark`, `GH_STACK_HYPERLINKS=0/1`; `GH_STACK_NO_UPDATE_NOTIFIER=1` disables optional release notices in v0.2.1.
These are session settings, not application secrets. Do not print existing authentication environment values.

## Exit codes and recovery

| Code | Meaning | Next step |
|---|---|---|
| 0 | Command returned successfully | Still verify PR creation/readiness/order and warnings. |
| 1 | Generic error | Diagnose the exact message; no speculative command switching. |
| 2 | Not in a stack / not found | Explicit checkout/import or correct the target; do not create an unrelated stack. |
| 3 | Rebase conflict | Stage the reported owner worktree; relevant `--continue` or `--abort`. |
| 4 | API failure | Check existing authentication, access, network/provider response. |
| 5 | Invalid arguments/flags | Consult this version's one relevant help section; never guess a replacement flag. |
| 6 | Branch belongs to multiple stacks | Select an unambiguous stack/PR or non-trunk branch. |
| 7 | Rebase already in progress | Complete/abort the existing recorded operation. |
| 8 | Stack mutation lock held | Identify the active owner/process; do not remove its lock. |
| 9 | Stack capability unavailable | Report endpoint/access/version limit; do not silently degrade to ordinary unlinked PRs. |
| 10 | Modify interrupted / recovery needed | Matching documented continue/abort, or finish pending submission. |

Use the [cookbook](cookbook.md#recover-without-guessing) for recovery context. Do not delete catalog/journal/lock files or broadly reset/clean someone else's work.