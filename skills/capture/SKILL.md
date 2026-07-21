---
name: capture
description: >-
  Quickly capture an idea or a vague bug as a new work item in the backlog,
  deliberately unrefined, to triage or refine later. Use for low-friction
  capture — "jot a ticket: …", "capture this", "file a quick bug for …",
  "note down …". A title is mandatory. Files with minimal ceremony to your
  tracker's default project. Requires a tracker adapter — see Integration.
  For a fully-fielded item, supply the fields in the prompt.
---

# Capture

Get an idea out of your head, into the backlog, fast. No refining now — that's a separate step.

## When to use

- Idea you don't want to refine now, just want tracked.
- Bug you spotted, need to look at later.
- Triggers: "jot a ticket", "capture this", "file a quick bug", "note down", "open a ticket so I don't forget".

Not for a fully-fielded item with sprint/parent/links — supply those explicitly, or refine after.

## Integration

Needs a **tracker adapter**:

- **create work item** — title, type, project → key + URL.

Only needs **resolve project metadata** if the user gives a sprint or parent.

## Input

- **Title** — mandatory. No title → ask once, stop. Never invent one.
- **Body** — optional, one line if given.

## Defaults — minimal by design

- **Project:** tracker default. The only pinned default.
- **Type:** `Bug` if it reads like a defect, else `Task`.
- **Everything else** (sprint, parent, repo, labels): unset → lands in backlog.

Don't ask for sprint/parent/repo proactively. Set one only if the user explicitly supplies it — then resolve its field ID via the adapter, never invent it.

## Workflow

1. Take the title (+ optional one-line body). No title → ask once, then stop.
2. Infer the type (Bug vs Task) from wording.
3. Preview in one line, file on OK: `Task on <project>: "webhook retries look flaky" → backlog. File it?`
4. Create via tracker adapter.
5. Report the key + URL.

## Hand-off

A captured item is intentionally thin. Refine it into acceptance criteria before implementing.

## Hard rules

- Never capture without a title. Ask once, then stop.
- One light confirmation before creating — nothing else.
- Never gather sprint/parent/repo proactively — only if explicitly supplied.
- Never invent tracker field IDs.
- One item per invocation unless the user asks for several.
