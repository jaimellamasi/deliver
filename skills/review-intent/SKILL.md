---
name: review-intent
description: >-
  Judges an open merge request from a business/intent perspective, not a technical one: is the
  linked ticket's acceptance criteria actually clear, what scenarios does the code handle that
  no AC or test covers, and for each error path — should it truly raise, or is it an expected
  case that shouldn't require action. Use when a ticket feels vague or you want a second opinion
  on whether the MR solves the right problem, separate from a technical review. Walks findings
  one at a time for confirm/debate/discard, then hands over the confirmed list (comment text +
  location) for the user to paste into the MR themselves. Requires a tracker adapter and a Git
  host CLI adapter — see Integration.
---

# Review Intent

Judge whether an MR solves the right problem — not whether the code is clean or the diff matches
the AC line-by-line (that's `review`). This skill exists because tickets are often vague, and a
technically solid diff can still: quietly resolve an ambiguity the wrong way, leave a real
scenario untested, or misjudge which errors should actually raise.

## When to use

Triggers: the ticket feels vague, or the user wants a second opinion on whether the MR solves the
right problem, separate from a technical/style review.

Not a replacement for `review` — run both if you want full coverage. Not for ticket-only input
(no MR yet) — this skill needs the diff to judge anything meaningfully; if a ticket is vague
before code exists, that's `refine`'s job.

## Integration

Needs two adapters, same as `review`:

- **Tracker** — read a work item by key (description, acceptance criteria).
- **Git host CLI** — read an MR's diff and description (e.g. `glab`). Never use WebFetch or raw
  API calls for this.

This skill never posts comments itself — `glab` has no draft/pending-review comment support
(only immediate posting), and using GitLab's raw API to work around that would break the "no raw
API calls" rule. Confirmed findings are handed to the user as text to paste into the MR
themselves.

Requires both a ticket and its MR — refuses to run on either alone.

## References

- `references/criticality-bar.md` — the bar for what counts as a critical finding worth
  surfacing vs. cosmetic noise to discard silently. Apply it before presenting anything.

## Core principles

- **Judge intent, not style.** Never duplicate `review`'s job — no AC-coverage checklist, no
  convention nitpicks, no clean-code/REST checks.
- **Filter ruthlessly before speaking.** Do the full analysis first, apply
  `references/criticality-bar.md`, and only ever present what's truly critical. State the count
  up front before walking through anything.
- **One finding at a time.** Present each proposed comment individually; wait for the user to
  confirm, debate, or discard it before moving to the next. Debate can change or eliminate a
  finding — this isn't a scripted checklist.
- **Never post anything.** This skill has no mechanism to post to the MR at all — it only ever
  collects confirmed findings for the user to paste in themselves, inline where a diff line is
  the natural anchor (uncovered scenarios, error-handling) or general/top-level (AC-vagueness
  findings that don't map to a single line).
- **Never invent an AC interpretation.** A vagueness finding must point to what the ticket
  actually says (or fails to say) — not to a reviewer's guess at what it "probably means."

## Workflow

```
Review Intent Progress:
- [ ] 1. Get the MR reference and its linked ticket — require both
- [ ] 2. Read the ticket → AC as written, not as assumed
- [ ] 3. Read the MR diff and its tests
- [ ] 4. Analyze: AC clarity, uncovered scenarios, error-handling judgment
- [ ] 5. Filter to critical-only via the criticality bar; state the count
- [ ] 6. Walk findings one at a time: confirm / debate / discard
- [ ] 7. Hand over the confirmed list for the user to paste into the MR
```

### 1. Get the MR and the ticket

Ask for both if not given. No linked ticket found → ask for the key directly. Refuse to proceed
on an MR with no ticket, or a ticket with no MR.

### 2. Read the ticket

Fetch AC via the tracker adapter, read it as written. Don't fill gaps with assumptions yet — that
comes in analysis, and every gap gets flagged, not silently resolved.

### 3. Read the MR

Fetch the diff and its tests via the Git host CLI adapter.

### 4. Analyze

- **AC clarity** — could the stated AC plausibly support more than one incompatible
  implementation? Where the diff picked one, is that choice actually justified by the ticket, or
  did it just guess?
- **Uncovered scenarios** — branches, edge cases, or error paths present in the diff with no
  corresponding test or AC mention.
- **Error-handling judgment** — for each error path touched: should this truly raise (a real
  failure), or is it an expected/normal case that shouldn't require raising an error at all?

### 5. Filter

Apply `references/criticality-bar.md`. Drop cosmetic findings entirely — don't mention they
existed. State the resulting count before showing anything: *"Found N things worth discussing."*
Zero critical findings → say so directly and stop.

### 6. Walk through findings

One at a time. For each: state the finding, the proposed comment text, and where it would go
(inline at a file:line, or general). Wait for the user's confirm / debate / discard. Debate may
revise the comment or drop the finding — don't move on until resolved.

### 7. Hand over

After all findings are walked through, present the final confirmed list — comment text plus
location for each — for the user to paste into the MR themselves. This skill never touches the
Git host to post anything.

## Hard rules

- Never post a comment to the MR — this skill has no posting mechanism, confirmed or not.
- Never batch-present without the one-at-a-time walkthrough.
- Never invent an AC interpretation not grounded in the ticket's actual text.
- Never surface a cosmetic finding — apply the criticality bar before presenting anything.
- Never state a finding without having read the actual diff and ticket.
- Never run on ticket-only or diff-only input — both are required.
- Never duplicate `review`'s job — no AC-coverage table, no style/convention/REST/clean-code
  checks.
