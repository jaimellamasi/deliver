# Item template

Section blocks for the work-item description. Pick what's relevant for the item — every section is optional except **Goal** and **Acceptance criteria**.

The worked examples below use a fictional external-API integration — "Acme Verify", a business-registration endpoint — to illustrate each block. They are examples, not a required shape — adapt or drop blocks that don't fit the item at hand.

## Goal

One or two sentences. State what the item changes about the running system, in terms a future engineer can verify against the deployed code.

```markdown
## Goal

Create Business API so `POST /v1/business-registry/:companyId/businesses`
actually registers the business at Acme Verify.
```

Avoid: "implement X" (vague), "item aims to" (preamble), "as part of the epic Y" (cross-reference).

## Scope

What's in, not what's out. List reference URLs verbatim — implementers will follow them.

```markdown
## Scope

Only the Create Business endpoint:
`POST {ACME_VERIFY_BASE_URL}/v1/business/business` →
`200 { id, requestId, timestamp }`. Status reads, removals and enrichment
enrollments are out of scope.

Reference:

- https://developer.acme-verify.example/docs/.../create-business
- https://developer.acme-verify.example/docs/.../authentication
```

## Domain changes

Type / enum / schema deltas. Mark BREAKING when changing public API contract.

```markdown
## Domain changes (breaking — V1 endpoint contract)

1. `industry` enum replaced with vendor's 21 values verbatim: `...`.
2. `numberOfEmployees` enum replaced with vendor's 7 values verbatim: `...`.
3. `taxId` becomes optional.
4. `dunsNumber` becomes required. Vendor policy: _"<verbatim quote>"_ (URL).
```

Each item: what changes, what to. When the change is non-obvious, append the **why** as a one-line citation.

## Request / response mapping

Table when integrating an external API. Verbatim field names, both sides.

```markdown
## Request payload mapping

| Domain field      | Vendor field         |
| ----------------- | -------------------- |
| `address.line1`   | `streetAddressLine1` |
| `address.country` | `country`            |
| `taxId`           | `einVatNumber`       |

`legalName`, `primaryName`, `industry`, `website` pass through with the
same name.

## Response handling

- `id` → persisted as `providerBusinessRef`
- `{ requestId, timestamp }` → persisted as `providerMetadata`
```

## Pre-call checks

Local validation, dedup, idempotency rules that fire BEFORE any external call.

```markdown
## Pre-provider duplicate checks

1. **Same-company:** an active row exists for `(companyId, provider='acme')`
   → 409 with reason `company_active`. Already implemented.
2. **Cross-company identity collision:** another company has an active row
   whose `legalName` or `primaryName` matches case-insensitively → 409 with
   reason `identity_collision`.
```

## Operational requirements

Anything that's a constraint on the implementation but not user-observable behaviour: auth, logging, retry policy, env config, alerting.

```markdown
## Operational requirements

- HTTP Basic auth: `Authorization: Basic base64(<APP_ID>:<APP_SECRET>)`.
- Credentials never appear in logs.
- No automatic retry on Create Business — service fee billed per registration.
- `ACME_VERIFY_APP_ID` / `_APP_SECRET` are optional in every environment.
  When absent, adapter logs a warning at boot and registration fails with 500.
- On non-2xx, the vendor status code, `error.code`, and `error.errors[0].message`
  are captured in logs against the originating `companyId`.
```

This section is the home for everything that "must be true" but isn't observable through the API. If a thing belongs here, it does NOT belong in AC.

## Future work

Known follow-ups that are deliberately out of scope. Flag racy / brittle bits the implementer might otherwise be tempted to fix in this item.

```markdown
## Future work

- Partial unique index on `lower(business_data->>'legalName')` in the
  next migration. Without it, the cross-company identity check is
  racy under concurrent registrations.
```

## Acceptance criteria

GIVEN/WHEN/THEN scenarios, customer-observable. One scenario per behaviour. Each scenario is a candidate integration test.

```markdown
## Acceptance criteria

### Scenario: Successful registration accepted by the provider

- **Given** no active business registration exists for the company at provider `acme`
- **And** provider credentials are configured
- **When** the company POSTs a valid registration
- **And** the provider responds with `200 { id, requestId, timestamp }`
- **Then** the API responds with 201 and a body containing the local business UUID,
  `pending` status, and the submitted-at timestamp
- **And** a row is persisted whose `providerBusinessRef` is the provider business `id`
  and whose `providerMetadata` carries the provider `requestId` and `timestamp`

### Scenario: Submission missing the required DUNS number

- **When** the company POSTs a registration without `dunsNumber`
- **Then** the API responds with 400
- **And** no call is made to the provider

### Scenario: Cross-company identity collision

- **Given** another company has an active business registration at provider `acme`
  whose `legalName` or `primaryName` matches case-insensitively
- **When** the company POSTs the registration
- **Then** the API responds with 409 and a body whose reason is `identity_collision`
- **And** no call is made to the provider
- **And** the response does not disclose the colliding company's identity
```

Common scenarios to cover for an integration item:

- Happy path (provider 200 → API 201, persisted state)
- Each schema-rejection case (missing required field, out-of-vocab enum)
- Each pre-call check (same-company dup, cross-company collision)
- Resubmission after soft-delete or rejection (allowed)
- Provider rejection (any non-2xx → API 500, no row persisted)
- Auth/secret guard (missing header → 401, no row, no provider call)
