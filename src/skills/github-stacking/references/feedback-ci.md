# Feedback and Distributed CI — Exact API Recipes

Use authenticated `gh` or equivalent supported GitHub tools. Substitute `OWNER`, `REPO`, `PR`, and observed IDs; never infer a thread ID from a comment's REST ID.
These are operational recipes; `stack-shepherding` owns disposition, authorization, timing, and completion rules.

## Request Copilot early

After verifying first-commit publication, the documented modern CLI request is:

`gh pr edit <PR> --repo <OWNER>/<REPO> --add-reviewer @copilot`.

For a CLI without that capability, the documented REST reviewer is **`copilot-pull-request-reviewer[bot]`**:

`gh api --method POST repos/<OWNER>/<REPO>/pulls/<PR>/requested_reviewers -f 'reviewers[]=copilot-pull-request-reviewer[bot]'`.

Inspect `gh api repos/<OWNER>/<REPO>/pulls/<PR>/requested_reviewers` and the review/timeline state before duplicate requests.
The documented REST re-review route is `.../requested_reviewers/rerequest` with the same reviewer array; use it when re-requesting a completed review, not when one is pending.
Automatic review on new pushes requires repository configuration; do not assume it. Request a fresh review for meaningful new heads when no automatic/pending request covers them.
Copilot replies do not solicit a bot conversation: GitHub states that replies are visible to humans, but Copilot does not read/respond to them. **You must resolve addressed threads yourself.**

## Enumerate all feedback

For each PR, paginate all of these, not just IDs after startup:

| Surface | Exact read |
|---|---|
| Inline comments | `gh api repos/<OWNER>/<REPO>/pulls/<PR>/comments --paginate` |
| Issue/PR comments | `gh api repos/<OWNER>/<REPO>/issues/<PR>/comments --paginate` |
| Review summaries/advisories | `gh api repos/<OWNER>/<REPO>/pulls/<PR>/reviews --paginate` |
| Check annotations | `gh api repos/<OWNER>/<REPO>/check-runs/<CHECK_ID>/annotations --paginate` |
| Check summary/details | `gh api repos/<OWNER>/<REPO>/check-runs/<CHECK_ID>` |

For thread state and actual GraphQL IDs, use this query with `gh api graphql --paginate`.
It supports outer thread pagination; if a thread has more than 100 comments, paginate that thread's comment connection separately before resolving it.

```graphql
query($owner: String!, $name: String!, $number: Int!, $endCursor: String) {
  repository(owner: $owner, name: $name) {
    pullRequest(number: $number) {
      reviewThreads(first: 100, after: $endCursor) {
        pageInfo { hasNextPage endCursor }
        nodes {
          id isResolved isOutdated viewerCanResolve
          comments(first: 100) {
            pageInfo { hasNextPage endCursor }
            nodes { id databaseId url body author { login } }
          }
        }
      }
    }
  }
}
```

Invocation: `gh api graphql --paginate -F owner='<OWNER>' -F name='<REPO>' -F number=<PR> -f query='<query above>'`.
Map REST `databaseId` to its containing GraphQL thread `id`; retain the map and thread's outstanding items in the ledger.
Do not treat `isOutdated` as resolved, and do not feed your own answer back into the new-request queue.

## Reply, resolve, and read back

1. Reply to the exact root review comment:
   `gh api --method POST repos/<OWNER>/<REPO>/pulls/<PR>/comments/<COMMENT_ID>/replies -f body='<verified fix/evidence and SHA>'`.
   Record the returned reply URL; retries first check whether it already exists.
2. Only after every item in the thread is addressed, resolve its observed GraphQL thread ID:
   `gh api graphql -f query='mutation($id: ID!) { resolveReviewThread(input: {threadId: $id}) { thread { id isResolved } } }' -f id='<THREAD_ID>'`.
3. Require no GraphQL `errors` and a returned `isResolved: true`; then read back:
   `gh api graphql -f query='query($id: ID!) { node(id: $id) { ... on PullRequestReviewThread { id isResolved viewerCanResolve } } }' -f id='<THREAD_ID>'`.
4. Mark closure only after readback confirms it. Permission/error/unresolved results remain pending; a replied-only row is not done.

No terminal resolution for `awaiting-user` or unaddressed items. Reopened/new feedback reactivates the ledger obligation.
For issue comments or standalone summary/advisory points, post an individual source-linked `gh pr comment <PR> --repo <OWNER>/<REPO> --body '<answer>'`.
Those surfaces have no review-thread resolution flag: record `not applicable`, not a fabricated mutation.

## Inspect distributed CI now

Observe all current-head providers, not a single aggregate workflow conclusion:

| Evidence | Exact read |
|---|---|
| PR head and rollup | `gh pr view <PR> --repo <OWNER>/<REPO> --json headRefOid,statusCheckRollup,mergeStateStatus,reviewDecision` |
| All check runs for head | `gh api 'repos/<OWNER>/<REPO>/commits/<HEAD_SHA>/check-runs?filter=all&per_page=100' --paginate` |
| Legacy/external statuses | `gh api repos/<OWNER>/<REPO>/commits/<HEAD_SHA>/statuses --paginate` |
| Workflow runs for head | `gh api 'repos/<OWNER>/<REPO>/actions/runs?head_sha=<HEAD_SHA>&per_page=100' --paginate` |
| Jobs for current attempt | `gh api 'repos/<OWNER>/<REPO>/actions/runs/<RUN_ID>/attempts/<ATTEMPT>/jobs?per_page=100' --paginate` |
| Individual job and failed steps | `gh api repos/<OWNER>/<REPO>/actions/jobs/<JOB_ID>` |
| Annotations, even while a run is active | `gh api repos/<OWNER>/<REPO>/check-runs/<CHECK_ID>/annotations --paginate` |
| Finished failed-job log when available | `gh api repos/<OWNER>/<REPO>/actions/jobs/<JOB_ID>/logs` (follows the authenticated log redirect) |

Current-attempt completed jobs/steps with `failure`, `timed_out`, or `action_required`, failure annotations, and external `failure`/`error` are actionable immediately.
Inspect cancellations as blockers when a required job did not run; do not equate intentional skipped/neutral results with a code defect.
An in-progress workflow can contain already failed jobs or steps. Record/route all known failures before applying idle backoff.
Whole-run `gh run view <RUN_ID> --log-failed` can be unavailable until completion; use job/step/annotation evidence now instead of waiting for it.
`gh api` follows the job-log redirect, but live log availability/permissions still vary; preserve failure evidence if that fetch is not yet available.
If even job logs are unavailable, keep the concrete failure queued and identify the unavailable evidence/provider; absence of logs is not absence of failure.

Group shared root causes for one owning-branch fix, but retain each failing check/job's obligation. After a push, re-evaluate latest SHA/attempt instead of deleting the remaining queue.
Dedup key: repository + branch + head SHA + run attempt + job/check/context + diagnostic identity. A changed attempt/head is not an already handled failure.
Never report green from stale successful runs, ignored API pages, inaccessible external providers, or rollup `in_progress`.

## Supported retry and missing-trigger recovery

- Re-run a verified transient/failed workflow attempt: `gh run rerun <RUN_ID> --repo <OWNER>/<REPO> --failed`.
- Trigger a workflow that actually has `workflow_dispatch`: `gh workflow run <WORKFLOW> --repo <OWNER>/<REPO> --ref <branch>`.
- A pull-request-only workflow is not dispatchable merely because its file exists. Inspect the trigger and the provider's documented recovery before acting.
- If a real new SHA is required and the caller authorizes it, make an explicit empty trigger commit on the owner, cascade, submit `--auto --open`, and verify the new run.
- Cap trigger attempts, preserve diagnostics, and escalate permissions/provider configuration; do not rerun a deterministic code failure or disable required checks.

## Official sources

- [Copilot review and CLI/REST requests](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/request-a-code-review).
- [Review-request API](https://docs.github.com/en/rest/pulls/review-requests#request-reviewers-for-a-pull-request).
- [Review-comment API](https://docs.github.com/en/rest/pulls/comments#reply-to-a-review-comment).
- [Workflow jobs and logs](https://docs.github.com/en/rest/actions/workflow-jobs).
- [Check runs and annotations](https://docs.github.com/en/rest/checks/runs).
- Resolution mutation fields were additionally verified by live **read-only** GitHub schema introspection; no live thread was mutated for this reference.