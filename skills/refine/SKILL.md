---
name: refine
description: Refines a work item into an implementation-ready spec with GIVEN/WHEN/THEN acceptance criteria. Use when the user provides a work-item key and asks to refine, groom, shape, draft, or "make ready" the item — typically with one or more reference URLs (vendor docs, design docs, parent epic). Output is pushed back to the same item. Requires a configured tracker adapter — see Integration.
---

# Refine

Turn a thin work item into a spec another engineer can implement with no follow-up questions: domain changes, external-system mapping, API behaviour — as GIVEN/WHEN/THEN scenarios that map straight to integration tests.

## When to use

Triggers: "refine", "groom", "shape", "draft AC", "make implementation-ready", or turning an epic stub into a sized item.

Not for filing new items from scratch — this assumes an existing item.

## Integration

Needs a **tracker adapter**:

- **read a work item by key** — description, parent, sprint, links.
- **update a work item** — replace description with the refined markdown.

"Read the item" / "push to the tracker" means: invoke that adapter. Reference URLs are fetched via your agent's web-fetch.

## Workflow

Iterative collaboration, not one-shot generation. Surface tradeoffs, get the user's call, push only when they say so.

```
Refine Progress:
- [ ] 1. Read the work item
- [ ] 2. Read the references
- [ ] 3. Explore the codebase
- [ ] 4. Draft sections in chat (not the tracker)
- [ ] 5. Surface tradeoffs — 1–3 per round
- [ ] 6. Iterate — never assume agreement
- [ ] 7. Convert AC to GIVEN/WHEN/THEN
- [ ] 8. Push — only on explicit OK
```

Step-by-step detail, extraction prompts, vocabulary-gap spotting: [workflow.md](references/workflow.md).

## Output structure

Pick what's relevant — not every item needs every section:

- **Goal** — one or two sentences
- **Scope** — in/out, cite reference URLs verbatim
- **Domain changes** — type/enum/schema deltas, flag breaking ones
- **Request/response mapping** — table, when integrating an external API
- **Pre-call checks** — validation, dedup, idempotency
- **Operational requirements** — auth, logging, retry policy, env config
- **QA Setup** — environment (default staging), endpoints, auth source, known fixtures; mark post-deploy-only as such
- **Future work** — known follow-ups, out of scope
- **Acceptance criteria** — GIVEN/WHEN/THEN, customer-observable only

Full template + worked examples: [template.md](references/template.md).

## Acceptance criteria style

Observable behaviour through the API — not an implementation checklist.

- One scenario per behaviour, headed `### Scenario: <short name>`
- **Given**/**When**/**Then**/**And** bullets
- Given = DB/external state; When = HTTP request; Then = response + persisted state
- Logging/env vars/retry policy → Operational requirements, not AC

```markdown
### Scenario: <short name>

- **Given** <DB / external state>
- **When** <HTTP request>
- **Then** <HTTP response status + body>
- **And** <persisted state>
```

Can't express it as Given/When/Then through the public API? It's probably Operational requirements.

## Tradeoffs to surface

Don't lock big calls unilaterally:

- Pass external errors through, or collapse to 5xx?
- Adopt vendor enum names directly, or maintain a translation map?
- Tighten our schema, or let the provider reject?
- Local dedup check, or rely on provider 409?
- Env config required everywhere, or optional?
- Defer or include: status reads, reconciliation, retries?

Full menu + framing language: [tradeoffs.md](references/tradeoffs.md).

## Hard rules

- Never paraphrase auth/error/schema sections of vendor docs — quote verbatim. Sparse web-fetch? Ask for the rendered page or credentials.
- Never reference sibling items or features in the description — the item must stand alone.
- Never put implementation details in AC. "Adds `FooAdapter` class" is wrong; "valid registration returns 201" is right.
- Never push without explicit confirmation. Show the draft, ask "push to `<KEY>`?", wait.
- Never invent fields or behaviour. Vendor docs silent? Flag as open, or ask.
- Never state something verified without having directly checked it.

## References

- [workflow.md](references/workflow.md) — step-by-step procedure, vendor-doc extraction prompts
- [template.md](references/template.md) — section-by-section template, worked examples
- [tradeoffs.md](references/tradeoffs.md) — design tradeoffs, framing language

> `template.md`/`tradeoffs.md` are built around external-API integration work. For frontend, infra, or pure-domain items, keep the GIVEN/WHEN/THEN discipline, drop what doesn't apply.
