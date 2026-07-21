# deliver

Claude Code plugin. Four skills, one workflow: idea → shipped, verified change.
Tool-agnostic — you wire your own tracker, APM, Git host via adapters.

## Workflow

```
capture  →  refine  →  implement  →  ⟨ship: PR · review · deploy⟩  →  qa
```

- **capture** — file a rough item in the backlog, fast.
- **refine** — turn it into a spec: GIVEN/WHEN/THEN acceptance criteria + QA Setup.
- **implement** — TDD it against the repo's own conventions, open a PR.
- **qa** — after deploy, validate against acceptance criteria via observability.

Shipping (PR → review → merge → deploy) is your own habit, not a skill.

Namespaced once installed: `/deliver:capture`, `/deliver:refine`, `/deliver:implement`, `/deliver:qa`.

## Principles

- Method over tooling — each skill isolates tool specifics in `## Integration`.
- The work item is the contract. Skills never call each other, only read/write the same item.
- Every outward-facing action (create, push, PR, live test) pauses for your confirmation.
- `implement` follows the target repo's conventions, not a hardcoded style.
- No skill claims "verified" without having actually checked.

## Adapters

| Adapter | Used by | Needs |
|---|---|---|
| Tracker | capture, refine, implement, qa | read/create/update item, resolve project metadata |
| Observability | qa | query logs/traces/metrics, list alerts |
| Git host CLI | implement | open PR, read comments, check CI (`gh`, `glab`) |
| Environment | qa | local env file with base URLs + auth tokens |

Bind these in your personal `CLAUDE.md`. See each skill's `## Integration` for specifics.

## Install

```bash
claude --plugin-dir /path/to/deliver
```

or copy skills in directly:

```bash
cp -r skills/* ~/.claude/skills/
```

## License

MIT — see [LICENSE](LICENSE).
