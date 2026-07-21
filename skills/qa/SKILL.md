---
name: qa
description: >-
  Performs QA on a deployed change the way a dedicated QA engineer would: reads the
  linked work item to learn what must be validated, drafts a test plan for user approval, then
  autonomously exercises the deployed system (GraphQL/REST endpoints the user supplies) and verifies
  behaviour through your observability platform. Uses the system's own mechanisms to set up test
  state; keeps assertions grounded in acceptance criteria, never in the implementation. Use after a
  change is deployed and the user asks to QA, test, validate, or verify a work item, feature, or fix
  against its acceptance criteria. Requires tracker and observability adapters — see Integration.
---

# QA

Validate a deployed change against its work item: set up each scenario using the system's own mechanisms, assert against acceptance criteria, verify through observability.

## Integration

Needs two adapters:

- **Tracker** — read a work item by key (description, acceptance criteria, linked docs).
- **Observability** — query logs, traces, metrics; list alerts/incidents over a window. Any APM works
  (Datadog, New Relic, Grafana/Loki, CloudWatch, Honeycomb, Splunk, …). Operations it must support:
  [observability-verification.md](references/observability-verification.md).

The deployed system under test (endpoints, environment, auth) comes from the user per run — not from the work item.

## Core principles

- **Assertions are AC-driven, never implementation-derived.** Expectations come only from the work
  item's AC, linked docs, and the API contract the user gives you. Don't read the implementation to
  decide what a scenario *should* return. AC gap blocking you? Surface it, ask — don't fill it from the
  code. Something you read while setting up state would change your expected outcome? Don't use it —
  that's an AC gap; ask instead of quietly folding it in.
- **Setup uses the system's own mechanisms.** Most scenarios need state that doesn't exist yet.
  Create it by calling the system's real APIs, running admin mutations, or using whatever tooling it
  exposes — the way a real user or integration would. Reading source to learn *how* to set up state is
  fine; it must never inform *what you expect back*.
- **Plan, then approval, then execute.** Never run a test request before the plan is approved. This
  touches deployed systems and real data.
- **Observability is always part of verification.** A correct-looking response isn't enough. For every
  scenario, confirm the change in observability and hunt for the unexpected (errors, retries, latency
  spikes, downstream failures, triggered alerts).
- **Autonomous by design.** The plan must run without further input. For each scenario, include the
  setup actions you'll perform. Need a credential, endpoint, or fixture you can't create yourself?
  Surface it in the plan, before approval — not mid-run.
- **Default environment is staging.** Only test elsewhere if the user explicitly names it.
- **Never state a claim as verified unless you directly inspected the evidence** (an observability
  query result, an actual response body). State assumptions as assumptions.

## Workflow

Copy this checklist and check items off as you go:

```
QA Progress:
- [ ] 1. Get the work-item key/URL from the user
- [ ] 2. Read the item → extract acceptance criteria
- [ ] 3. Gather the test surface + fill tool gaps with the user
- [ ] 4. Draft the test plan → present → get explicit approval
- [ ] 5. Execute each scenario: request + observability verification
- [ ] 6. Deliver the QA report — ask whether a saved artifact is wanted, and in what format
- [ ] 7. Only after the report exists: offer to file/refine items from it
```

### 1. Get the work item

Ask the user for the work-item key or URL. Don't infer it.

### 2. Read the work item

Fetch via your tracker adapter. Read description, acceptance criteria, comments; follow links to
design/product docs. Distil into a concrete, numbered list of AC. Item vague? Say so, ask the user to
clarify — don't infer AC from the implementation.

Check for a **QA Setup** section (from the `refine` skill). If present, treat it as the starting point
for step 3 — only ask the user for what it leaves unresolved (e.g. fixtures marked "confirm post-deploy").

### 3. Gather the test surface

Check your environment adapter (e.g. a `staging.env`-style file) first for base URL(s) and auth
token(s). Combine with anything the item's QA Setup already states. Ask the user only for what's
still missing:

- **Environment**: defaults to staging — confirm the base URL from the adapter file or QA Setup;
  only ask if neither has it.
- **Interface**: the GraphQL/REST endpoint(s) involved. Ask for a sample query/mutation or request
  body and the auth token.
- **Setup actions**: what state each scenario needs, and whether you can create it via the system's
  own APIs/tooling — or the user must supply a pre-existing fixture (ID, account, etc.).
- **Observability scope**: service name(s), `env` tag, correlation key (request id, trace id, company
  id, user id). See [observability-verification.md](references/observability-verification.md).

List every gap explicitly as something the user must provide.

### 4. Draft and validate the test plan

Build the plan from [test-plan-template.md](references/test-plan-template.md). Every AC maps to ≥1
scenario; include happy-path, edge, negative, and regression scenarios. Each scenario must run
unattended: exact request, exact expected result, the observability check.

Present the full plan, wait for explicit approval. Incorporate edits. Don't proceed on silence or a
vague "looks fine" if scenarios still need inputs.

### 5. Execute

Run approved scenarios one by one: issue the request, capture the full response, run the
observability verification over the matching window — confirm expected signals and scan for
unexpected ones. Record pass/fail with concrete evidence (response snippet, log lines, trace ids).
Don't fix code — report what you find.

### 6. Report

Deliver per [test-plan-template.md](references/test-plan-template.md): overall verdict, per-scenario
results with evidence, defects (severity + reproduction), AC coverage, what still needs manual QA.

**Before producing any other artifact, ask whether the user wants the report saved and in what
format** — a repo file, a `/tmp` file, an item comment, a wiki page, or just inline. Don't assume.

### 7. Items come from the report, not before it

Report first → review → then file or refine items from it. Only offer item creation once the report
exists and the user has seen it; confirm scope first — which findings become items, how they split
by repo/owner, new items vs. refining existing ones. Filing is outward-facing — get explicit
go-ahead before creating.

## References

- [test-plan-template.md](references/test-plan-template.md) — test plan and QA report formats.
- [observability-verification.md](references/observability-verification.md) — how to verify behaviour and detect anomalies via your APM.
