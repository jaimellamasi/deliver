---
name: refine
description: Refines an existing issue's scope and behavioral acceptance criteria. Use when the user asks to refine a ticket, clarify requirements, or draft acceptance examples before implementation.
---

# Refine

Make one issue precise enough to implement and verify within its agreed scope. Finish with
clear acceptance criteria and no unresolved decisions blocking implementation. For a batch,
apply this boundary to each issue.

## Boundaries

Clarify required behavior and scope. Read and link accepted architecture and shared contracts.
Surface unresolved decisions affecting shared contracts, system structure, or significant
security, reliability, or migration choices; resolve them explicitly before treating them as
requirements. Leave local code structure, file choices, and execution planning to implementation.

## Integration

Read `$XDG_CONFIG_HOME/deliver/integrations.md` (default `~/.config/deliver/integrations.md`).
Task choices and repository instructions override these defaults. Ask only for gaps needed
now; preserve saved providers, accounts, and destinations. If unconfigured, offer
`deliver-setup` or resolve the required choices for this run.

Needs a tracker adapter to read the issue, comments, and linked context, and update its description.

## Understand

Read the issue, relevant comments, and linked decisions. Separate accepted requirements from
proposals and future work; surface material conflicts. Read related issues when they define
contracts or dependencies this issue needs. Link shared sources rather than copying them.

Identify the actors, expected behavior, boundaries, and exclusions. Inspect repository
instructions and existing behavior when needed to resolve a concrete ambiguity. Verify
relevant external contracts; preserve exact agreed names and values without inventing missing
fields or treating implementation assumptions as requirements.

## Clarify

Distinguish rules, examples illustrating them, and unanswered questions. Use representative
examples to expose relevant failures, boundaries, authorization, concurrency, and compatibility.
Preserve scope; recommend splits or follow-ups rather than silently restructuring the work.

Ask focused questions when answers change scope or correctness; continue refining unaffected
behavior. An unknown expected outcome is a question. Distinguish blocking decisions from
choices the implementer can safely make.

## Acceptance criteria

Integrate criteria into the existing issue, replacing ambiguous or redundant wording.
Use Given/When/Then when it clarifies behavior:

- **Given:** relevant state, actor, and preconditions.
- **When:** one meaningful action or event.
- **Then:** verifiable outcome, including state that must remain unchanged.

Adapt to the issue's boundary: UI behavior, API contracts, background processing, or
operational outcomes. Keep agreed technical and nonfunctional constraints where relevant.
Use simple checklists or example tables when clearer; impose no scenario count or test framework.
Leave routine implementation steps out of the criteria.

Include QA setup only when useful: known environment, access references, and fixtures needed
to verify the behavior. Flag unavailable prerequisites; never include credentials.

## Review and deliver

Check coverage of the agreed scope, verifiable outcomes, and compatibility with linked
contracts. Keep the issue concise; do not require a fixed template or claim that examples
have passed tests.

Show the draft and obtain confirmation before updating the tracker. Before writing, fetch
the latest description and preserve unrelated content and metadata. Prefer targeted edits;
if the adapter requires replacing the body, merge changes into the latest version. Read back
the result; inspect an ambiguous write before retrying.

Report the draft or updated issue link and any blocking questions. Do not claim readiness
while a blocking decision remains unresolved.
