# deliver

Six standalone skills, one workflow: idea → shipped, verified change.
Tool-agnostic — you wire your own tracker, APM, Git host via adapters.

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
| Environment | qa | local env file with base URLs + auth tokens |

Document these bindings in the target repository's `AGENTS.md` or your personal agent configuration.
See each skill's `## Integration` for specifics.

## Install

For Codex, copy or symlink individual skill directories from `skills/` into
`~/.agents/skills/` for personal use, or a target repository's `.agents/skills/`
for use in that repository. Keep each `SKILL.md` and its supporting files together.

Invoke a skill in Codex CLI or the IDE extension with `$capture`, `$refine`,
`$implement`, `$review`, `$review-intent`, or `$qa`.

See the [Codex skills documentation](https://learn.chatgpt.com/docs/build-skills)
for discovery and installation details.

## License

MIT — see [LICENSE](LICENSE).
