# deliver

A tool-agnostic, human-in-the-loop software-delivery workflow, packaged as a Claude Code **plugin**. Its skills take a unit of work from product idea to verified change — capture, refine, implement, QA — wired to your own tracker and observability tools via adapters.

## The workflow

```
capture  →  refine  →  implement  →  ⟨ship: PR · review · deploy⟩  →  qa
```

- **capture** — quickly file a new work item in the backlog, deliberately unrefined, to refine later.
- **refine** — turn a thin work item into an implementation-ready spec with GIVEN/WHEN/THEN acceptance criteria, plus a `## QA Setup` section so QA isn't guessed after the fact.
- **implement** — read the refined item, confirm the target repo, plan, branch, build under TDD following the repo's own conventions, verify against the repo's gate, and open a PR.
- **qa** — black-box validation of the *deployed* change against its acceptance criteria, defaulting to staging, verified through observability.

The shipping step (commit → PR → review → merge → deploy) is intentionally left to your own Git/CI habits; `implement` ends at an opened PR and `qa` resumes once the change is live.

Once installed, the skills are namespaced under the plugin: `/deliver:capture`, `/deliver:refine`, `/deliver:implement`, `/deliver:qa`.

## Design principles

1. **Methodology over tooling.** The value is the *method* — the refinement discipline, the TDD-to-PR flow, the black-box QA stance. Tool specifics (which tracker, which APM, which Git host) are isolated in a single `## Integration` section per skill so they can be swapped without touching the method.
2. **The work item is the contract — for the spec.** `refine` writes the spec onto the work item; `implement` reads it. Neither skill calls the other; they communicate through the shared item. `qa` is a parallel black-box stage that reads the same item for acceptance criteria while getting its runtime surface (endpoints, environment) elsewhere.
3. **No inter-skill coupling.** A skill never names or invokes a sibling skill. Composition lives in your agent's router, not hardcoded inside skills.
4. **Human approval gates are deliberate.** Each skill that touches an outward-facing system (creating an item, pushing a spec, opening a PR, exercising a deployed system) pauses for explicit confirmation.
5. **Follow the repo, not the skill.** `implement` carries no style checklist. Code style and simplicity defer to the target repo's own conventions and your agent's global guidance.
6. **Ground claims in evidence.** No skill states something as "verified" or "confirmed" without having directly inspected the source (code, vendor doc, API response, observability query). Assumptions are surfaced as assumptions.

## Adapting to your tools

Every tool dependency lives in a skill's `## Integration` section. To adopt these skills you wire adapters:

| Adapter | Used by | Operations to expose |
|---|---|---|
| **Tracker** | capture, refine, implement, qa | resolve project metadata · read a work item by key · update a work item · create a work item |
| **Observability** | qa | query logs / traces / metrics · list alerts & incidents over a window |
| **Git host CLI** | implement | open a PR/MR, read comments, check CI (e.g. `gh`, `glab`) |
| **Environment** | qa | a local credentials/env file supplying base URLs and auth tokens |

"Wire an adapter" can mean an MCP server, a CLI the agent can call, or a documented manual step. The skills are written against the *operations*, not any specific tool — declare your concrete bindings in your personal `CLAUDE.md`.

## Install

As a Claude Code plugin, from a local clone:

```bash
claude --plugin-dir /path/to/deliver
```

Or copy individual skills into your agent's skills directory:

```bash
cp -r skills/* ~/.claude/skills/
```

Then configure the adapters described in each skill's `## Integration` section.

## License

MIT — see [LICENSE](LICENSE).
