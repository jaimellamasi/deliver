# Item template

Section blocks for the work-item description. Pick what's relevant for the item — every section is optional except **Goal** and **Acceptance criteria**.

The placeholders below illustrate each block without assuming a particular product, provider, or data model. Fill them with the item's actual requirements; adapt or drop blocks that don't fit.

## Goal

One or two sentences. State what the item changes about the running system, in terms a future engineer can verify against the deployed code.

```markdown
## Goal

<Observable change and who or what benefits.>
```

Avoid: "implement X" (vague), "item aims to" (preamble), "as part of the epic Y" (cross-reference).

## Scope

What's in, not what's out. List reference URLs verbatim — implementers will follow them.

```markdown
## Scope

<Included operation(s) and explicit exclusions that bound this item.>

Reference:

- <authoritative specification or contract URL>
- <additional relevant reference URL, if needed>
```

## Domain changes

Type / enum / schema deltas. Mark BREAKING when changing public API contract.

```markdown
## Domain changes

1. <Field, type, enum, or schema change: current shape → required shape.>
2. <Whether the change breaks an existing contract, with the relevant source or decision.>
```

Each item: what changes, what to. When the change is non-obvious, append the **why** as a one-line citation.

## Request / response mapping

Table when integrating an external API. Verbatim field names, both sides.

```markdown
## Request payload mapping

| Domain field | External field | Transformation, if any |
| ------------ | -------------- | ---------------------- |
| <field>      | <field>        | <documented mapping>   |

## Response handling

- <External response field> → <required handling or persistence rule>
```

## Pre-call checks

Local validation, dedup, idempotency rules that fire BEFORE any external call.

```markdown
## Pre-call checks

1. **<Validation or duplicate condition>:** <expected result and whether an external call is allowed>.
2. **<Additional condition, if applicable>:** <required behavior>.
```

## Operational requirements

Anything that's a constraint on the implementation but not user-observable behaviour: auth, logging, retry policy, env config, alerting.

```markdown
## Operational requirements

- Authentication: <required mechanism and configured credential source>.
- Credentials never appear in logs.
- Retry policy: <agreed retry conditions, limits, and duplicate-effect handling>.
- Configuration: <required settings and agreed behavior when they are absent>.
- Diagnostics: <relevant failures, fields, and correlation identifiers>.
```

This section is the home for everything that "must be true" but isn't observable through the API. If a thing belongs here, it does NOT belong in AC.

## Future work

Known follow-ups that are deliberately out of scope. Flag racy / brittle bits the implementer might otherwise be tempted to fix in this item.

```markdown
## Future work

- <Deferred improvement, its impact, and why it is separate from this item.>
```

## Acceptance criteria

GIVEN/WHEN/THEN scenarios, customer-observable. One scenario per behaviour. Each scenario is a candidate integration test.

```markdown
## Acceptance criteria

### Scenario: <Successful operation>

- **Given** <relevant initial state>
- **And** <required configuration or external preconditions>
- **When** <request with the inputs that matter>
- **Then** <contract-defined response status and body>
- **And** <required persisted state or side effect>

### Scenario: <Validation or conflict condition>

- **Given** <state that triggers the condition, if needed>
- **When** <request that exercises the condition>
- **Then** <contract-defined response and permitted side effects>
```

Common scenarios to cover for an integration item:

- Happy path: expected response and persisted state from the agreed contract
- Each relevant schema-rejection case
- Each specified pre-call check
- Repeated submissions or retries, where relevant
- External failure or rejection, using the item's agreed error-handling policy
- Authentication and authorization failures, where applicable
