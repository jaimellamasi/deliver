---
name: qa
description: >-
  Performs black-box QA on a deployed change the way a dedicated QA engineer would: reads the
  linked work item to learn what must be validated, drafts a test plan for user approval, then
  autonomously exercises the deployed system (GraphQL/REST endpoints the user supplies) and verifies
  behaviour through your observability platform. Use after a change is deployed and the user asks to
  QA, test, validate, or verify a work item, feature, or fix against its acceptance criteria.
  Deliberately does not read application source code, to avoid implementation bias. Requires tracker
  and observability adapters — see Integration.
---

# QA

Validate a deployed change against its work item the way an independent QA engineer would: from the
outside, against acceptance criteria, never against the implementation.

## Integration

This skill needs two adapters:

- **Tracker** — read a work item by key (description, acceptance criteria, linked docs).
- **Observability** — query logs, traces, and metrics, and list alerts/incidents over a time window.
  Any APM works (Datadog, New Relic, Grafana/Loki, CloudWatch, Honeycomb, Splunk, …); wire whichever
  your team runs. See `references/observability-verification.md` for the operations it must support.

The deployed system under test (its endpoints, environment, auth) is supplied by the user per run —
it is not read from the work item.

## Core principles

- **Black box. Never read application source code.** Do not grep, open, or reason about resolvers,
  use-cases, business logic, or test files of the thing under test. Reading the implementation biases
  QA toward what the code *does* instead of what the item *requires*. Expectations come only from:
  the work item and its acceptance criteria, linked product/design docs, and the API contract the user
  provides (endpoint, request/response shape, sample payloads). If you need the contract, ask the
  user for it — do not derive it from the repo.
- **Plan, then get approval, then execute.** Never run a single test request before the user has
  approved the test plan. Testing touches deployed systems and real data.
- **Observability is always part of verification.** A response looking correct is not enough. For every
  scenario, confirm the change in your observability platform — and actively hunt for *unexpected*
  behaviour (errors, exceptions, retries, latency spikes, downstream failures, triggered alerts) the
  response hides.
- **Autonomous by design.** The plan must be executable by you without further input. If a scenario
  needs data, credentials, an endpoint, or a tool you do not have, surface that gap *in the plan* and
  ask the user to provide it before approval — not mid-run.
- **Default environment is staging.** Only test against another environment (e.g. sandbox) if the
  user explicitly names it. Never assume sandbox.
- **Never state a claim as verified or confirmed unless you directly inspected the evidence** (an
  observability query result, an actual response body). State assumptions explicitly as assumptions.

## Workflow

Copy this checklist into your response and check items off as you go:

```
QA Progress:
- [ ] 1. Get the work-item key/URL from the user
- [ ] 2. Read the item → extract acceptance criteria (no code)
- [ ] 3. Gather the test surface + fill tool gaps with the user
- [ ] 4. Draft the test plan → present → get explicit approval
- [ ] 5. Execute each scenario: request + observability verification
- [ ] 6. Deliver the QA report — ask whether a saved artifact is wanted, and in what format
- [ ] 7. Only after the report exists: offer to file/refine items from it
```

### 1. Get the work item

Ask the user for the work-item key or URL. Do not infer it.

### 2. Read the work item

Fetch it via your tracker adapter. Read the description, acceptance criteria, and comments. Follow links
to design/product docs when they clarify expected behaviour. Distil the item into a concrete, numbered
list of **acceptance criteria** (what must be true). If the item is vague, say so and ask the user to
clarify the expected behaviour — do not fill the gap by reading the code.

Also check for a **QA Setup** section (produced by the `refine` skill). If present, it already states
the environment, endpoints, auth source, and known fixture data — treat it as the starting point for
step 3 and only ask the user for what it leaves unresolved (e.g. fixtures marked "confirm post-deploy").

### 3. Gather the test surface

Before asking the user anything, check for a configured local credentials/environment file (your
environment adapter — e.g. a `staging.env`-style file your `CLAUDE.md` points to). If found, use it as
the default source for base URL(s) and auth token(s). Combine this with anything already stated in the
item's **QA Setup** section. Then ask the user only for what's still missing:

- **Environment**: defaults to staging (see Core principles) — confirm the base URL from the adapter
  file or QA Setup section; only ask if neither has it.
- **Interface**: the GraphQL or REST endpoint(s) involved, depending on the item. Ask the user to
  state these in the prompt; request a sample query/mutation or request body and the auth token.
- **Test data**: any IDs, accounts, companies, fixtures the scenarios need.
- **Observability scope**: service name(s), `env` tag, and a correlation key (request id, trace id,
  company id, user id) so behaviour can be located. See `references/observability-verification.md`.

List any gap explicitly as something the user must provide.

### 4. Draft and validate the test plan

Build the plan using `references/test-plan-template.md`. Every acceptance criterion maps to at least
one scenario; include happy-path, edge, negative, and regression scenarios. Each scenario must be
concrete enough to run unattended (exact request + exact expected result + the observability check).

Present the full plan and **wait for explicit approval**. Incorporate the user's edits. Do not proceed
on silence or a vague "looks fine" if scenarios are still missing inputs.

### 5. Execute

Run the approved scenarios one by one. For each: issue the request, capture the full response, then run
the observability verification (`references/observability-verification.md`) over the matching time
window — confirm the expected signals *and* scan for unexpected ones. Record pass/fail with concrete
evidence (response snippet, log lines, trace ids). Do not fix code; report what you find.

### 6. Report

Deliver the QA report per the format in `references/test-plan-template.md`: overall verdict, per-scenario
results with evidence, defects (severity + reproduction), acceptance-criteria coverage, and anything
that still needs manual QA.

**Before producing any other artifact, ask the user whether they want the report saved and in what
format** — e.g. a markdown file in the repo, a file under `/tmp`, a comment on the work item, a wiki
page, or just inline in the chat. Do not assume. The report is the primary deliverable and the source
of truth.

### 7. Items come from the report, not before it

Do **not** jump straight to creating work items. The default flow is **report first → review → then
file or refine an item based on it**. Only offer item creation once the report exists and the user
has seen it, and confirm scope first: which findings become items, how they split by
repository/owner, and whether to open new items or refine existing ones. Filing items is an
outward-facing action — get explicit go-ahead before creating them.

## References

- `references/test-plan-template.md` — test plan and QA report formats.
- `references/observability-verification.md` — how to verify behaviour and detect anomalies via your APM.
