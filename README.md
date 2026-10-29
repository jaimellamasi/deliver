# deliver

Six workflow skills and a setup skill: idea → shipped, verified change.
Choose your tracker, APM, and Git host once with `deliver-setup`; keep those choices when switching agents.

## Workflow

```
capture  →  refine  →  implement  →  review  →  ⟨ship: merge · deploy⟩  →  qa
```

- **capture** — file a rough item in the backlog, fast.
- **refine** — turn it into a spec: GIVEN/WHEN/THEN acceptance criteria + QA Setup.
- **implement** — TDD it against the repo's own conventions, open a PR.
- **review** — check an open MR against its item's AC and the repo's own conventions.
- **review-intent** — complement review by checking whether an MR solves the right business problem.
- **qa** — after deploy, validate against acceptance criteria via observability.

Shipping (merge → deploy) is your own habit, not a skill.

## Principles

- Method over tooling — each skill isolates tool specifics in `## Integration`.
- The work item is the contract. Skills never call each other, only read/write the same item.
- Every outward-facing action (create, push, PR, live test) pauses for your confirmation.
- `implement` follows the target repo's conventions, not a hardcoded style.
- No skill claims "verified" without having actually checked.

## Adapters

| Adapter | Used by | Needs |
|---|---|---|
| Tracker | capture, refine, implement, review, review-intent, qa | read/create/update item, resolve project metadata |
| Observability | qa | query logs/traces/metrics, list alerts |
| Git host CLI | implement, review, review-intent | open PR, read diff/comments, check CI (`gh`, `glab`) |
| Environment | qa | references for base URLs and authentication |

An adapter is a binding to an existing connector, MCP tool, or CLI; no adapter code is required.
See each skill's `## Integration` for specifics.

## Setup once

After installation, run `deliver-setup`. It discovers available tools, helps you choose
accounts and destinations, checks access with read-only operations, and saves personal
defaults in `~/.config/deliver/integrations.md` (or
`$XDG_CONFIG_HOME/deliver/integrations.md` if set). The file belongs to you, independently
of the agent and project. Only connection choices and credential references are saved, never secrets.

The workflow skills read these defaults on each invocation. Explicit task choices and
repository instructions take precedence. Configure only what you need now; run setup
again to add or change an integration. Existing manually configured adapters still work.
When switching agents, keep the same file. Connections and authentication may need setting
up in the new agent; your provider, account, and destination choices stay the same.

## Install

Install the individual directories from `skills/` using your agent's skill installation
mechanism. Keep each `SKILL.md` and its supporting files together.

Invoke `deliver-setup`, `capture`, `refine`, `implement`, `review`, `review-intent`, or `qa`
using your agent's skill invocation syntax.

## License

MIT — see [LICENSE](LICENSE).
