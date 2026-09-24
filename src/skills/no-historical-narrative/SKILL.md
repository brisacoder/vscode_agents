---
name: no-historical-narrative
description: The binding "present state only, no history" rule that every agent applies whenever it writes or reviews a document — a design spec, functional spec, implementation plan, technical spec, README, docstring, architecture diagram, or code-review report. Load before producing or reviewing any prose artifact. Forbids historical commentary, investigation narratives ("we first thought X, then discovered Y, then changed to Z"), archaeological decision logs, changelog-style prose embedded in a design document, and conversational framing ("as we can see", "it turns out", "after some digging"). A document must describe only the current architecture, the current mathematical or data models, the current implementation plan, and the current design decisions and rationale — never the path taken to reach them. Version history belongs in git and CHANGELOG.md, not in the body of a specification or review. In review mode this rule is a filable finding; in write mode it is a hard constraint on every artifact produced.
user-invocable: false
context: inline
---

# Present State Only — No Historical Narrative

Every document you write or review must describe **the current state of the
world**: the architecture as it is now, the data and mathematical models as they
are now, the implementation plan as it stands now, and the design decisions and
their rationale as they hold now. It must **not** narrate how that state was
reached.

LLMs and agents have a strong tendency to be verbose about the history of a
document — to recount the investigation that led to a conclusion, to preserve
every superseded idea, to write as if narrating a conversation. This is a defect.
The reader of a design spec wants to know what the design *is*, not to follow an
archaeological expedition through the hundreds of decisions that produced it.

This rule is binding for every prose artifact: design specs, functional specs,
implementation plans, technical specs, READMEs, docstrings, architecture-diagram
notes and title blocks, and code-review reports. It is not a stylistic
preference.

## Forbidden by default — historical commentary

Do not write the history of the document or of the thing it describes:

- "We originally thought the cache was write-through, then investigated and found
  it is write-back, but then realized …" — state only that the cache is
  write-back and why.
- "Previously this used a global lock; it was later changed to a per-shard lock."
  — describe the per-shard lock as the current design. The old design is not part
  of the current design.
- "In an earlier version …", "this used to …", "we migrated away from …" — unless
  documenting a migration *is itself the current task* (a migration guide), the
  prior state does not belong in the body.

## Forbidden by default — investigation narratives

Do not narrate the process of discovery:

- "After some digging we found …", "it turns out that …", "on closer inspection …",
  "we traced the bug to …" — present the conclusion directly.
- "We first hypothesized A, ruled it out, then B, ruled that out, and settled on
  C." — document C. The ruled-out branches are not the current thinking.
- Blow-by-blow accounts of debugging sessions, spikes, or proofs-of-concept that
  led to the design.

## Forbidden by default — archaeological decision logs

A design document records the decisions that *stand*, with their rationale — not a
running log of every decision ever considered:

- Keep the decision and its *current* justification (this is legitimate and
  valuable — "we use a per-shard lock because contention profiling showed …").
- Drop the chronology, the abandoned alternatives that add no present value, and
  the "we debated X vs Y for a long time" framing. If an alternative genuinely
  informs the current design, state the trade-off in the present tense, not as a
  story.

## Forbidden by default — conversational framing

Documents are not transcripts:

- "As we can see …", "let's now turn to …", "you might be wondering …", "recall
  that earlier we said …" — remove. Write declarative, present-tense prose.
- First-person-plural journey narration ("our journey", "we then explored") —
  remove.

## Where history *does* belong

History is not worthless — it is simply stored in the right place:

- **git history** records how the code changed, commit by commit.
- **`CHANGELOG.md`** records user-facing changes between released versions.
- **A migration guide or ADR (Architecture Decision Record)**, when that document
  is *explicitly* the artifact requested, may record superseded decisions — but
  even then, scoped to that document's purpose, not smeared across every spec.

Never reconstruct these into the body of a design spec, README, docstring, or
review report.

## In review mode

When reviewing a document (a spec, README, docstring, or another agent's review
report), treat any historical commentary, investigation narrative, archaeological
decision log, or conversational framing as a **filable finding**. Quote the
offending passage, state which category it falls into, and rewrite it in
present-tense, present-state form. Preserving a superseded design or a discovery
narrative "for context" is not a valid defense — the context is the current state.

## In write mode

Every artifact you produce states only current thinking. Before declaring a
document ready, re-read it and delete any sentence whose subject is the past of
the document itself rather than the present of the thing it describes. If you
catch yourself writing "originally", "previously", "we then", "it turns out", or
"after investigating", stop and rewrite in the present tense.
