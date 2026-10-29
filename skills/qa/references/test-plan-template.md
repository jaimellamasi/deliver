# Test plan & QA report formats

## Test plan

Present this to the user for approval before executing anything.

```markdown
# QA Test Plan — <ITEM-KEY>: <title>

**Environment:** <env + base URL>
**Interface(s):** <GraphQL/REST endpoint(s)>
**Auth:** <how requests are authenticated>
**Observability scope:** service=<service> env=<env> correlation=<key>

## Acceptance criteria (from the item)
1. <AC-1>
2. <AC-2>

## Inputs still needed from you
- <endpoint / token / test data / fixture> — required for scenarios <ids>
  (omit this section if nothing is missing)

## Scenarios
### S1 — <short title>
- **Covers:** AC-1
- **Type:** happy | edge | negative | regression
- **Setup:** <actions to execute via the system's own mechanisms to create the required state; note any fixture the user must supply>
- **Action:** <exact request — operation + payload/params, or curl line>
- **Expected response:** <status / fields / values>
- **Observability check:** <what to confirm + what anomaly to rule out>

### S2 — ...
```

Rules:
- Every acceptance criterion is covered by ≥1 scenario.
- Include at least one negative scenario (invalid input, unauthorized, missing data) and, where the
  change touches existing behaviour, at least one regression scenario.
- A scenario the agent cannot run unattended is not done — its missing input goes under "Inputs still
  needed from you".

## QA report

Deliver this after execution.

```markdown
# QA Report — <ITEM-KEY>: <title>

**Verdict:** ✅ PASS | ❌ FAIL | ⚠️ BLOCKED
**Environment:** <env>   **Date:** <date>

## Summary
<2–3 sentences: what was tested, the outcome, the headline defects.>

## Scenario results
| ID | Scenario | Result | Evidence |
|----|----------|--------|----------|
| S1 | <title>  | ✅/❌/⚠️ | <response snippet · trace id · log link> |

## Defects
### D1 — <title> (severity: blocker | major | minor)
- **Scenario:** S<n>
- **Expected:** <...>
- **Actual:** <...>
- **Reproduction:** <exact request>
- **Evidence:** <response / log lines / trace id>

## Acceptance-criteria coverage
- AC-1 — ✅ covered & passing (S1)
- AC-2 — ❌ failing (S3 → D1)

## Needs manual QA / out of scope
- <anything that couldn't be validated autonomously and why>
```
