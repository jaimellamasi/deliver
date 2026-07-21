---
name: implement
description: Implement a work item end to end. Use when the user provides a work-item key and wants it implemented. Fetches the item, understands the acceptance criteria, drives TDD implementation following repo conventions, verifies against the repo's gate, and opens a PR linked to the item. Requires a configured tracker adapter — see Integration.
---

# Implement

Take a refined work item and drive it to a review-ready pull request: read the acceptance criteria, agree a plan, implement under TDD following the repository's own conventions, verify against the repo's gate, and open a PR linked to the item. The acceptance criteria are the contract — implement what they require and no more.

## When to use

Triggers:

- The user provides a work-item key (e.g. `PROJ-1234`) and asks to implement, build, or code it.

Do NOT use this skill to refine or shape a work item — it assumes the acceptance criteria already exist. If the item has none, stop and ask for it to be refined first.

## Integration

This skill needs a **tracker adapter** to **read a work item by key** (its description, acceptance criteria, and links). Wherever this skill says "fetch the work item", it means "invoke that adapter." Everything else — Git, TDD, the verification gate, the PR — is tool-neutral and uses the repo's own conventions.

## Workflow

Copy this checklist and check off each step as you go:

```
Implement Progress:
- [ ] 1. Read the work item → fetch it; confirm it has acceptance criteria (else stop)
- [ ] 2. Plan               → files to touch, AC→test mapping, stated assumptions; wait for confirmation
- [ ] 3. Branch             → fresh from the repo's default branch
- [ ] 4. Implement          → TDD from the acceptance criteria, following repo conventions
- [ ] 5. Verify             → run the repo's full gate; everything green
- [ ] 6. Open a PR          → only on explicit confirmation; linked to the item
```

### 1. Read the work item

Fetch the work item via your tracker adapter. This skill assumes the item is already refined with a description and acceptance criteria; if it has none, stop and ask for it to be refined first rather than implementing against an absent contract. Ask clarification questions only for decisions that genuinely block progress. Where something is ambiguous at the implementation level, make a reasonable decision and state it explicitly.

Determine the target repository from the work item (e.g. a dedicated repository field, or stated explicitly in the description). If the item doesn't say, confirm the repository with the user before touching any file.

### 2. Plan

Present a brief plan and wait for confirmation before writing any code. The plan states the target repository, the files and modules to touch, how each acceptance criterion maps to one or more tests, and any assumptions made on ambiguous points.

### 3. Branch

Always start fresh from the repository's default branch. Follow the branch convention documented in the repo (README, `AGENTS.md`/`CLAUDE.md`); otherwise use conventional branching:

```bash
git checkout <default-branch>
git fetch && git pull
git checkout -b <type>/<KEY>-meaningful-brief-description
```

- `<type>`: `feat`, `fix`, or `refactor` — inferred from the item type
- Branch name is lowercase, hyphen-separated, brief but descriptive

### 4. Implementation

Follow the repository's own conventions — `AGENTS.md`/`CLAUDE.md` and the patterns in neighbouring files. Drive the work with strict TDD from the acceptance criteria (red-green-refactor); each acceptance criterion maps to one or more tests.

Before running any codegen or install command, read the repo's `AGENTS.md`/`CLAUDE.md` and `README.md` for the documented toolchain (e.g. a codegen script, a specific install command). Use that. Never improvise a workaround (e.g. a `NODE_PATH` hack, a manual `yarn install`) that bypasses the repo's own scripts — these can silently rewrite generated files or lockfiles.

Code style and simplicity are governed by the repo's conventions and your agent's global guidance — do not hardcode a style checklist here.

### 5. Verify

Before declaring done, run the repository's full verification gate — lint, typecheck, build, and tests. Discover the exact command from the repo's conventions (`AGENTS.md`/`CLAUDE.md`, `package.json` scripts, Makefile, or README); many repos expose a single aggregate script for this. Everything must pass. Do not open a PR on a red gate — fix and re-run.

### 6. Open a pull request

Use your configured Git-hosting CLI adapter (e.g. `gh`, `glab`) for all remote operations — opening the PR/MR, reading comments, checking CI status. Never use WebFetch or raw API calls for these. Your `CLAUDE.md` states which CLI is your adapter.

Opening a PR is an outward-facing action: with the gate green, present a summary of the diff and **wait for explicit confirmation** before opening it. Once confirmed:

- Title references the item (e.g. `<KEY>: <brief summary>`).
- Body summarises what changed and maps the work back to the acceptance criteria.
- Link the PR to the item per the repo/Git convention.

Report the PR URL. This is the skill's exit point — review, merge, and deploy stay with the user.

## Hard rules

- **Never implement a work item with no acceptance criteria.** Stop and ask for it to be refined first.
- **Never write code before the target repository is confirmed** — read it from the work item; if absent, confirm with the user.
- **Never commit to the default branch.** Always work on a feature branch.
- **Never improvise codegen/install workarounds.** Use the repo's documented toolchain (`AGENTS.md`/`CLAUDE.md`/`README.md`); never bypass it with ad hoc commands.
- **Never declare done on a red gate.** Lint, typecheck, build, and tests must pass before a PR.
- **Never use WebFetch/raw API for Git-hosting operations.** Use your configured CLI adapter.
- **Never open a PR without explicit confirmation.** It is an outward-facing action.
- **Never state a claim as verified or confirmed unless you directly inspected the evidence** (the actual file, command output, or upstream response). State assumptions explicitly as assumptions.
- **Follow the repository's conventions over personal preference.**
- **State every implementation-level assumption explicitly** rather than deciding silently.
