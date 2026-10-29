---
name: implement
description: Implements a scoped ticket or work item. Use when the user asks to implement, build, or code an existing issue.
---

# Implement

Implement one issue within its agreed scope. Finish with the required behavior implemented,
the final diff reviewed, and the repository's verification gate green.

## Simplicity first

Choose the simplest complete solution. Reuse existing patterns; add abstractions only for
a concrete need. Judge simplicity across the whole flow, including indirection and external
calls. Remove obsolete code and scaffolding left by earlier approaches.

## Integration

Read `$XDG_CONFIG_HOME/deliver/integrations.md` (default `~/.config/deliver/integrations.md`).
Task choices and repository instructions override these defaults. Ask only for gaps needed
now; preserve saved providers, accounts, and destinations. If unconfigured, offer
`deliver-setup` or resolve the required choices for this run.

Needs a tracker adapter to read the issue, comments, and linked context.

## Understand

Read the issue, relevant comments, architecture, and shared contracts. Read related tickets
only when they define behavior or contracts this implementation depends on. Separate accepted
decisions from proposals; preserve agreed scope.

Use the current Git working tree unless the issue or user identifies another repository.
Resolve conflicting or unclear targets before editing. Use the assigned checkout/worktree
and preserve unrelated work.

Read repository instructions, trace the affected entry point, and inspect a comparable
feature and its tests. Reuse established models, validation, and error handling.

Missing or unclear acceptance criteria: ask only about decisions affecting scope or
correctness, and continue unaffected work. Never invent required behavior.

## Plan

Before editing, briefly state the behavior, affected files, acceptance-criteria coverage,
verification commands, and material assumptions. Reuse an agreed plan; follow repository
planning conventions. Pause for unresolved scope or design decisions, or when the user
requests plan approval.

## Implement

Follow repository conventions and the documented toolchain for setup, installs, and codegen.
Explain material departures from existing architecture before introducing them.

For behavior changes, use small red–green–refactor cycles. Confirm the failing test exposes
the intended missing behavior, then make the simplest change that passes. Cover acceptance
criteria with meaningful behavior tests, reusing existing fixtures and coverage; no fixed
test count per criterion.

## Review and verify

Review the final diff and follow the repository's required review process. Fix actionable
findings, then run the required package/repository gate on the final code. Confirm checks
actually ran; failed or unavailable required checks mean the work is incomplete.

Finish with a short account of the change, checks and results, and any remaining blockers.
