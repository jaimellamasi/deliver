---
name: implement
description: Implement a work item end to end. Use when the user provides a work-item key and wants it implemented. Fetches the item, understands the acceptance criteria, drives TDD implementation following repo conventions, verifies against the repo's gate, and opens a PR linked to the item. Requires a configured tracker adapter — see Integration.
---

# Implement

Take a refined work item to a review-ready pull request: read the acceptance criteria, agree a plan, implement under TDD following the repo's own conventions, verify against the repo's gate, open a PR linked to the item. AC is the contract — build what it requires, no more.

## When to use

Triggers: user gives a work-item key and asks to implement, build, or code it.

Not for refining/shaping an item — this assumes AC already exists. No AC? Stop, ask for it to be refined first.

## Integration

Needs a **tracker adapter** to **read a work item by key** (description, AC, links). Everything else — Git, TDD, the gate, the PR — is tool-neutral, uses the repo's own conventions.

## Workflow

```
Implement Progress:
- [ ] 1. Read the work item → confirm AC exists, else stop
- [ ] 2. Plan               → files, AC→test mapping, assumptions; wait for confirmation
- [ ] 3. Branch             → fresh from the default branch
- [ ] 4. Implement          → TDD from AC, repo conventions
- [ ] 5. Verify             → full gate, everything green
- [ ] 6. Open a PR          → only on explicit confirmation, linked to the item
```

### 1. Read the work item

Fetch via tracker adapter. No AC → stop, ask for refine first, don't implement against an absent contract. Ask only what genuinely blocks progress; state assumptions on everything else.

Target repo comes from the item (a dedicated field, or stated in the description). Not stated? Confirm with the user before touching any file.

### 2. Plan

Present a brief plan, wait for confirmation before writing code: target repo, files/modules to touch, AC→test mapping, assumptions on ambiguous points.

### 3. Branch

Always fresh from the default branch. Follow the repo's documented convention (README, `AGENTS.md`/`CLAUDE.md`), or:

```bash
git checkout <default-branch>
git fetch && git pull
git checkout -b <type>/<KEY>-meaningful-brief-description
```

`<type>`: `feat`, `fix`, or `refactor`, inferred from the item type. Lowercase, hyphen-separated, brief.

### 4. Implementation

Follow the repo's own conventions — `AGENTS.md`/`CLAUDE.md`, neighbouring files. Strict TDD from AC (red-green-refactor); each AC maps to one or more tests.

Before codegen/install, read the repo's documented toolchain and use it. Never improvise a workaround (a `NODE_PATH` hack, manual `yarn install`) — it can silently rewrite generated files or lockfiles.

Style and simplicity: repo conventions + your agent's global guidance, not a checklist here.

### 5. Verify

Run the repo's full gate — lint, typecheck, build, tests. Discover the command from the repo's own conventions. Everything passes. Red gate → fix and re-run, no PR.

### 6. Open a pull request

Use your Git-hosting CLI adapter (`gh`, `glab`) for all remote ops — never WebFetch or raw API calls.

Outward-facing: present the diff with a green gate, wait for explicit confirmation. Then:

- Title references the item (`<KEY>: <brief summary>`).
- Body summarises the change, maps back to AC.
- Link the PR to the item.

Report the PR URL. This is the exit point — review, merge, deploy stay with the user.

## Hard rules

- Never implement a work item with no acceptance criteria — ask for refine first.
- Never write code before the target repo is confirmed.
- Never commit to the default branch.
- Never improvise codegen/install workarounds — use the repo's documented toolchain.
- Never declare done on a red gate.
- Never use WebFetch/raw API for Git-hosting operations — use your CLI adapter.
- Never open a PR without explicit confirmation.
- Never state something verified without having directly inspected the evidence.
- Follow the repo's conventions over personal preference.
- State every implementation-level assumption explicitly.
