# Design tradeoffs to surface

When an item integrates an external provider/API, these are the recurring design questions to raise explicitly with the user during refinement. Don't lock any of them unilaterally. An item with no external integration won't hit most of these — surface only the ones that apply.

For each: state the choice, the tradeoff in one sentence, and a recommendation if you have one.

## Provider abstraction integrity

**The question:** when an external provider rejects our call (validation error, conflict, rate limit), do we pass that through with detail to our caller, or collapse everything past the schema gate to a generic 5xx?

- **Pass-through (typed):** caller sees actionable error from provider — UI can render "EIN format invalid for country GB". Couples our public API to the provider's error vocabulary. Hard to swap providers.
- **Collapse to 5xx (recommended for new integrations):** abstraction stays clean. Every provider rejection becomes an alert, which forces us to tighten our schema iteratively until rejections only happen on outages. Worse first-occurrence UX, much cleaner steady state.

## Vocabulary alignment (enums, field names)

**The question:** the provider has its own enum vocabulary that doesn't match ours. Adopt theirs, or maintain a translation map?

- **Adopt vendor vocabulary:** breaking change to our public API but zero translation overhead. Right when there's only one provider and unlikely to change.
- **Translation map in adapter:** stable public API, but every new vendor enum value triggers a code change in two places. Right when multi-provider is real or imminent.

Default recommendation: adopt vendor vocabulary unless multi-provider is concrete.

## Validation responsibility

**The question:** the provider validates field X (format, length, conditional requirements). Do we mirror it in our schema, or let the provider reject?

- **Mirror in our schema:** caller sees 400 fast, no provider call. Preserves provider abstraction. Cost: our schema drifts from theirs over time.
- **Let provider reject:** less code, but every novel provider rule is a 5xx alert.

Default: mirror the rules the provider documents explicitly (required fields, enum values). Don't try to mirror format regexes — those drift fastest.

## Local vs. provider-side dedup

**The question:** the provider returns 409 on duplicate. Do we add a pre-flight DB check, or rely on the provider 409?

- **Pre-flight DB check:** clean 409 to caller without a billed provider call (when the provider charges). Doesn't catch cross-tenant duplicates the provider knows about and we don't.
- **Provider 409:** simpler, but billed call wasted on duplicates and abstraction leak.

Default: pre-flight when the provider charges per call OR the duplicate condition is fully expressible in our DB.

## Env config strictness

**The question:** are vendor credentials required at startup, or optional with a graceful degradation path?

- **Required (fail fast):** misconfigured prod can't deploy. Forces correct config but blocks staged rollouts.
- **Optional (degrade gracefully):** prod can deploy with empty creds and the endpoint returns 5xx until cutover. Required for staged rollouts.

Default: optional, with a boot-time warning and a clear 500 on call when missing.

## Retry policy

**The question:** retry transient provider failures, or fail fast?

- **Retry:** improves apparent reliability for idempotent reads.
- **No retry:** mandatory for billed writes (every retry costs money) and non-idempotent operations.

Default: no retry on writes, especially when the provider charges per call. Add retry only for idempotent GETs in a follow-up item.

## Scope cuts

**The question:** what's in V1 and what's a follow-up?

Common cuts to surface:
- **Status reads / reconciliation:** usually a separate item — needs a polling loop and idempotency story
- **Update / delete operations:** rarely V1 unless explicitly required
- **Webhook receivers:** treat as a separate item — auth model and DB schema are different
- **Multi-provider support:** start with one, add the registry pattern when the second provider lands

Default: cut aggressively. A V1 with one endpoint shipping is better than a V2 with three blocked.

## How to frame these in chat

```
Two calls before I push:

1. **<Question A in one line>** — recommend X because <one-line tradeoff>. Y if <when Y is right>.
2. **<Question B in one line>** — recommend X because <one-line tradeoff>.

Once those are settled I'll write the final item.
```

Limit to 2–3 per round. More than that and the user can't keep state.
