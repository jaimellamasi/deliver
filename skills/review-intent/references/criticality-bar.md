# Criticality bar — what to surface, what to drop

Caveman rules. Apply before presenting any finding. If it fails the bar, drop it silently —
never mention it existed.

## General rule

If a reasonable senior engineer looking at this MR would not block merge over it, it's cosmetic.
Drop it.

## AC clarity

- Critical: the ambiguity could plausibly lead to a *different, incompatible* implementation.
  Example — ticket doesn't say what happens on duplicate submission, code silently picked one
  behavior.
- Not critical: wording is awkward but the intent is obvious. Missing nice-to-have detail.
  Cosmetic ticket formatting.
- Test: could a second engineer, reading the same ticket, have reasonably built something
  meaningfully different? If no, drop it.

## Uncovered scenarios

- Critical: the untested path is reachable in production and produces a materially different
  outcome than the tested paths (different persisted state, different response, different
  side effect).
- Not critical: untested log statement. Untested branch provably equivalent to a tested one.
  Defensive code for a case that structurally cannot occur.
- Test: if this exact untested branch ran right now in prod, would the outcome differ from what
  tests already assert? If no, drop it.

## Error handling

- Critical: the classification is actually wrong — something raises but shouldn't (breaks a
  legitimate flow), or something is silently swallowed but should surface (hides a real
  failure).
- Not critical: a debatable style choice between two equally-valid error strategies the repo
  doesn't already standardize on.
- Test: does the current behavior actively hide a bug, or actively break a case that's supposed
  to work? If it's just "I'd have done it differently," drop it.

## No artificial cap

Don't cap the count to look tidy. If the bar above is applied honestly, the count naturally
stays small. A large count on a large MR is itself signal — surface it as-is, don't hide items to
make the list shorter.
