# Refine workflow

Step-by-step procedure for taking a work item from "thin idea" to "implementation-ready spec".

## 1. Read the work item

Fetch the item via your tracker adapter, reading its full content. Capture: summary, current description (often empty), parent epic, sprint, links. The parent epic often holds the project-level "why" — fetch it too if the item description gives no context.

## 2. Read the references

The user usually provides one or more URLs. Fetch each in parallel with your agent's web-fetch capability.

For vendor API docs, ask for **everything verbatim**:
- Endpoint method, path, base URL
- Request body: every field name, type, required/optional, enum values, format constraints (regex, max length, country codes, phone formats)
- Response body: every field with type, especially the field carrying the resource id
- Auth scheme: token endpoint, grant type, header format
- Error model: status codes, error envelope shape, rate-limit headers

If web-fetch returns sparse content (JS-rendered API explorers often do), say so explicitly to the user and ask them to paste the rendered content.

## 3. Explore the codebase

- Find the touched module(s) — wherever the affected code lives for this feature
- Read the current adapter / port / service / domain files
- Read existing patterns in **sibling features** to understand conventions, but DO NOT cite them in the item itself
- Read `AGENTS.md`/`CLAUDE.md` and any in-repo design docs
- Check schema files for the persisted shape

The goal is to know what changes, what stays, and what conventions to follow — without leaking those internals into the item as cross-references.

## 4. Identify gaps

Compare vendor contract against current domain:

| Gap                        | Surface to user                                         |
|----------------------------|---------------------------------------------------------|
| Field renames              | Add to mapping table                                    |
| Required ↔ optional flips  | Domain schema change, flag as breaking                  |
| Enum mismatches            | Decide: adopt vendor vocabulary OR translate in adapter |
| Nested ↔ flat structure    | Mapper denormalises in adapter                          |
| Auth model assumptions     | Verify against vendor docs                              |
| Error model                | Decide error mapping policy                             |

These are **decision points** for the user, not unilateral calls. Frame each as a tradeoff (see `tradeoffs.md`).

## 5. Draft sections in chat

Present the draft in chat first, NOT in the tracker. Use the section blocks from `template.md`. Keep it tight — drafts should be terse and skimmable, never padded with preamble.

Common over-corrections to avoid:
- Cross-references to other features ("mirrors the X integration") — drop them
- Implementation details in AC — move to Operational requirements
- Verbose preambles ("This item aims to…") — cut to the goal

## 6. Surface tradeoffs

Each round, surface 1–3 design questions before iterating further. Phrase each with:
- The choice (option A vs option B)
- The tradeoff in one sentence
- A recommendation if you have one

Stop adding sections until the user weighs in. Big-picture calls (error mapping policy, abstraction integrity, scope cuts) should never be locked unilaterally.

## 7. Convert AC to GIVEN/WHEN/THEN

Once the user agrees on shape, convert each behaviour bullet to a scenario. Rules:
- One scenario per behaviour
- Each scenario starts with a Given (state of the world) — for stateless validation, you can start with When
- Whens describe HTTP requests with the body shape that matters
- Thens describe HTTP response status, body fields, AND persisted state
- Use **And** for additional preconditions or assertions, NOT for chaining unrelated steps

If a scenario keeps growing past 8 bullets, it probably needs splitting into two.

## 8. Push to the tracker

Only after explicit confirmation ("push it", "looks good, push", etc.): update the work item's description via your tracker adapter with the full markdown. Confirm the URL back to the user.

## When to re-iterate

After pushing, the user often raises one more refinement (e.g. "make AC GIVEN/WHEN/THEN", "drop the cross-references"). Treat each as a follow-up update to the same item — re-push the full description, don't append.
