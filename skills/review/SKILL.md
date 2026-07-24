---
name: review
description: >-
  Reviews an open merge request against its linked work item's acceptance criteria and the
  target repo's own conventions. Checks AC coverage, test quality, and correctness — not
  personal style preference. Use when the user gives an MR/PR URL or number and asks to review
  it, check if it's ready to merge, or get a second opinion before merging. Reports findings in
  chat; only posts to the MR on explicit confirmation. Requires a tracker adapter and a Git host
  CLI adapter — see Integration.
---

# Review

Check whether an open merge request actually does what its work item asks — no more, no less
— using the target repo's own conventions as the bar for quality. Complements `qa` (which
validates *deployed* behaviour): this reviews the *diff*, before merge. For a business/intent
judgment instead of a technical one — is the AC actually clear, what scenarios aren't tested,
should this error path truly raise — see `review-intent`.

## When to use

Triggers: user gives an MR/PR URL or number and asks to review it, check if it's mergeable, or
sanity-check it before merging.

Not for reviewing your own uncommitted working diff — that's a separate local review flow.

## Integration

Needs two adapters:

- **Tracker** — read a work item by key (description, acceptance criteria).
- **Git host CLI** — read an MR's diff, description, comments, and CI status (e.g. `gh`, `glab`).
  Never use WebFetch or raw API calls for this.

## References

- `references/rest-principles.md` — REST checklist. Load it whenever the diff adds or modifies a
  REST endpoint, and verify the changed endpoint(s) against it during step 4 (Assess).
- `references/clean-code-principles.md` — clean code checklist (naming, functions, duplication,
  complexity, etc), language-agnostic. Load it for every review and verify the diff against it
  during step 4 (Assess).

## Core principles

- **Grounded in the item's AC, not personal taste.** Findings trace back to what the acceptance
  criteria require, or to the target repo's own documented conventions (`AGENTS.md`/`CLAUDE.md`,
  neighbouring code) — never to a reviewer's stylistic preference.
- **Report, don't fix.** This skill finds and describes issues; it does not push commits to the
  branch.
- **Posting to the MR is outward-facing.** Approving, requesting changes, or leaving inline
  comments happens only after the user confirms — the review report itself is free to share in
  chat.
- **Never state something as broken or missing without having read the actual diff.** No
  guessing from the MR title or description alone.

## Workflow

```
Review Progress:
- [ ] 1. Get the MR reference and the linked work-item key
- [ ] 2. Read the item → acceptance criteria
- [ ] 3. Read the MR: diff, description, CI status
- [ ] 4. Map diff → AC coverage; flag gaps, extras, convention breaks, test quality
- [ ] 5. Present the review report
- [ ] 6. Only on confirmation: post comments / approve / request changes via the Git host CLI
```

### 1. Get the MR and the item

Ask for the MR URL/number if not given. Resolve the work-item key from the MR title/branch name
or ask for it directly.

### 2. Read the work item

Fetch AC via the tracker adapter. Vague or missing AC? Say so — review against what's stated, and
flag the gap rather than inventing expectations.

### 3. Read the MR

Fetch the diff, description, and CI status via the Git host CLI adapter. Note the target repo's
own conventions before judging style.

### 4. Assess

- **AC coverage** — does the diff satisfy every AC? Anything implemented that no AC asked for?
- **Test quality** — do tests assert the AC's behaviour, or just exercise the code path?
- **Convention adherence** — does the diff follow the repo's own patterns, not a generic ideal?
- **Correctness** — obvious bugs, edge cases the diff misses, error handling gaps.
- **REST principles** — if the diff adds or modifies a REST endpoint, check it against
  `references/rest-principles.md`.
- **Clean code** — check the diff against `references/clean-code-principles.md` (naming,
  functions, duplication, complexity).

### 5. Report

Verdict (ready to merge / changes needed / blocked), AC-coverage table, findings by severity
(blocking/minor), CI status. Present in chat first.

### 6. Act only on confirmation

Posting inline comments, approving, or requesting changes via the Git host CLI happens only after
the user says so.

## Hard rules

- Never post to the MR (comment/approve/request-changes) without explicit confirmation.
- Never judge against personal style — only against AC and the repo's own conventions.
- Never state a finding as fact without having read the actual diff or CI output.
- Never fix code from this skill — report findings, let `implement` or the author fix them.
