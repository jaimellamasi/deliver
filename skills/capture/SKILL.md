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

Get an idea or a half-seen bug out of your head and into the backlog **now**, without refining it. The point is speed: a title, optionally a line of context, filed — so it's tracked and you can pick it up later. Refinement is a separate step, done when you choose to work the item.

## When to use

- You have an idea you don't want to refine right now and just want it tracked.
- You spotted a bug or oddity you need to look at later.
- Triggers: "jot a ticket", "capture this", "file a quick bug", "note down", "open a ticket so I don't forget".

Do NOT use this to file a carefully-fielded item with sprint/parent/links — for that, supply the fields explicitly, or refine the item afterwards. This skill optimises for capture, not completeness.

## Integration

This skill needs a **tracker adapter** exposing one operation for the common path:

- **create work item** — title, type, project → returns the key + URL.

Only if the user explicitly supplies a sprint or parent does it also need **resolve project metadata** (to find that field's identifier). Wherever this skill says "create the work item", it means "invoke that adapter."

## Input

- **Title** — mandatory (the one-line summary). If absent, ask once and stop. Never invent one.
- **Body** — optional. A single line of context if the user gave any; otherwise leave it empty.

## Defaults — minimal by design

- **Project:** your tracker's default project. The only pinned default.
- **Type:** inferred from wording — `Bug` if it reads like a defect, otherwise `Task`.
- **Everything else** (sprint, parent, repository, labels): left unset → lands in the backlog.

Do NOT interrogate the user for sprint/parent/repository — that friction defeats capture. Set one of those only if the user **explicitly supplies it** in the prompt; in that case resolve its field identifier from project metadata via your adapter, and never invent an ID.

## Workflow

1. **Take the title** (+ optional one-line body). No title → ask once, then stop.
2. **Infer the type** (Bug vs Task) from the wording.
3. **Preview in one line and file on OK** — e.g. `Task on <project>: "webhook retries look flaky" → backlog. File it?` Creating a work item is outward-facing, so wait for a single confirmation — but ask nothing else.
4. **Create** the work item via your tracker adapter (project default, inferred type, title, optional body).
5. **Report** the key + URL.

## Hand-off

A captured item is intentionally thin. When you come to work it, refine it into acceptance criteria first — capture is the inbox, refinement is the separate step that makes it implementation-ready.

## Hard rules

- **Never capture without a title.** Ask once, then stop.
- **One light confirmation before creating** — it is outward-facing. But confirm only; do not interrogate for fields.
- **Never gather sprint/parent/repository proactively.** Honour them only if explicitly supplied.
- **Never invent tracker field IDs.** Resolve from metadata, and only when a field was supplied.
- **One item per invocation** unless the user explicitly asks for several.
