---
name: refine
description: Refines a work item into an implementation-ready spec with GIVEN/WHEN/THEN acceptance criteria. Use when the user provides a work-item key and asks to refine, groom, shape, draft, or "make ready" the item — typically with one or more reference URLs (vendor docs, design docs, parent epic). Output is pushed back to the same item. Requires a configured tracker adapter — see Integration.
---

# Refine

Turn a thin work-item idea into a spec another engineer can implement without follow-up questions. The output reads like a contract: what changes in the domain, how external systems map onto it, what behaviour the API must exhibit — captured as GIVEN/WHEN/THEN scenarios that translate directly into integration tests.

## When to use

Triggers:

- User provides a work-item key and says "refine", "groom", "shape", "draft AC", or "make implementation-ready"
- User asks to convert an idea / parent-epic stub into a sized item

Do NOT use this skill to file new items from scratch with no target — the workflow assumes an existing item to update.

## Integration

This skill needs a **tracker adapter** exposing two operations:

- **read a work item by key** — its description, parent, sprint, and links.
- **update a work item** — replace its description with the refined markdown.

Wherever this skill says "read the item" or "push to the tracker", it means "invoke that adapter." External reference URLs are fetched with your agent's web-fetch capability.

## Workflow

This is an **iterative collaboration**, not a one-shot generation. Surface tradeoffs, get the user's call on each, only push to the tracker when they say so.

Copy this checklist and check off each step as you go:

```
Refine Progress:
- [ ] 1. Read the work item        → via your tracker adapter
- [ ] 2. Read the references        → vendor docs, parent epic, design docs
- [ ] 3. Explore the codebase       → current state of touched modules, conventions
- [ ] 4. Draft sections in chat     → present in chat, NOT in the tracker
- [ ] 5. Surface tradeoffs          → 1–3 design questions per round, expect pushback
- [ ] 6. Iterate                    → fold user feedback; never assume agreement
- [ ] 7. Convert AC to GIVEN/WHEN/THEN
- [ ] 8. Push                       → update the item (only on explicit OK)
```

For each step in detail, including what to extract from external docs and how to spot vocabulary gaps: see [workflow.md](references/workflow.md).

## Output structure

The item description is built from these blocks. Pick what's relevant — not every item needs every section.

- **Goal** — one or two sentences, no preamble
- **Scope** — what's in and what's explicitly out; cite reference URLs verbatim
- **Domain changes** — type/enum/schema deltas, called out as breaking when they are
- **Request/response mapping** — table when integrating an external API
- **Pre-call checks** — local validation, dedup, idempotency rules
- **Operational requirements** — auth, logging, retry policy, env config
- **QA Setup** — environment (default staging), endpoints, auth source, and known fixture data for whoever QAs this once deployed; mark anything only knowable post-deploy as "confirm post-deploy"
- **Future work** — known follow-ups that are out of scope but worth tracking
- **Acceptance criteria** — GIVEN/WHEN/THEN scenarios, customer-observable only

The full template with worked examples lives in [template.md](references/template.md).

## Acceptance criteria style

Acceptance criteria describe **observable behaviour through the API surface** — what an integration test would assert. They are NOT a checklist of implementation tasks.

- One scenario per behaviour, headed `### Scenario: <short name>`
- Bullet list of `**Given**`, `**When**`, `**Then**`, `**And**`
- Givens describe DB / external state; Whens describe HTTP requests; Thens describe HTTP response + persisted state
- Implementation/operational concerns (logging, env vars, retry policy) belong in their own **Operational requirements** section, NOT as AC bullets

Skeleton (full worked examples live in [template.md](references/template.md)):

```markdown
### Scenario: <short name>

- **Given** <DB / external state>
- **When** <HTTP request>
- **Then** <HTTP response status + body>
- **And** <persisted state>
```

If a behaviour can't be expressed as a Given/When/Then through the public API, it probably belongs under **Operational requirements**, not AC.

## Tradeoffs to surface

Don't lock big-picture calls unilaterally. Common decision points worth raising explicitly with the user:

- **Provider abstraction integrity**: pass external errors through, or collapse to 5xx?
- **Vocabulary alignment**: adopt the vendor's enum names directly, or maintain a translation map?
- **Validation responsibility**: tighten our schema vs. let the provider reject?
- **Local vs. provider-side dedup**: pre-flight DB check or rely on provider 409?
- **Env config**: required everywhere, or optional with empty-prod allowed?
- **Scope cuts**: status reads, reconciliation, retries — defer or include?

The full menu of recurring tradeoffs with framing language is in [tradeoffs.md](references/tradeoffs.md).

## Hard rules

- **Never paraphrase auth, error, or schema sections of vendor docs.** Quote field names verbatim. If web-fetch returns sparse content, ask the user for the rendered page or credentials to access the schema directly.
- **Never reference sibling items or other features** in the description (e.g. "mirrors the X integration"). The item must be self-contained — implementers won't have that context six months later.
- **Never include implementation details in AC.** "Adds `FooAdapter` class" is wrong; "submitting a valid registration returns 201" is right.
- **Never push to the tracker without explicit confirmation.** Show the draft in chat, ask "push to `<KEY>`?", wait.
- **Never invent fields or behaviour.** If the vendor docs don't specify it, flag as an open item or ask the user.
- **Never state a claim as verified or confirmed unless you directly inspected the source** (the actual code, vendor doc, or API response). State assumptions explicitly as assumptions.

## References

- [workflow.md](references/workflow.md) — step-by-step procedure with extraction prompts for vendor docs
- [template.md](references/template.md) — section-by-section template with worked examples
- [tradeoffs.md](references/tradeoffs.md) — design tradeoffs and how to frame them

> Note: `template.md` and `tradeoffs.md` are framed around **external-API integration** work (the kind with vendor docs, request/response mappings, and provider error policies). Treat them as worked examples — for frontend, infrastructure, or pure-domain items, keep the GIVEN/WHEN/THEN discipline and drop the integration-specific sections that don't apply.
