# Observability verification

Observability is part of every scenario's verification: confirm the change produced the *expected*
signals, and actively rule out *unexpected* ones a correct-looking response can hide (silent errors,
retries, downstream failures, latency regressions, triggered alerts).

This document is written against **operations**, not a specific tool. Map each operation to your APM —
Datadog, New Relic, Grafana/Loki, CloudWatch, Honeycomb, Splunk, or whatever your team runs.

## Contents
- Load your platform's guidance first
- Establish the search context
- Confirm expected behaviour
- Hunt for unexpected behaviour
- Capture evidence

## Load your platform's guidance first

If your observability tool ships query guidance, conventions, or an agent skill, load it before
querying so searches are well-formed (correct query syntax, index/scope names, time-window semantics).
Skip only if it is already loaded this session.

## Establish the search context

Pin down before querying:
- **Time window** — bracket it tightly around when you issued the request.
- **Scope** — `service`, `env` tag, and a correlation key (request id, trace id, company id, user id,
  or a unique value from your payload). Get these from the user in the plan's "Observability scope".

## Confirm expected behaviour

- **Logs for the request** — query logs filtered by service + env + correlation key over the window.
  Confirm the expected log lines, status, and emitted fields.
- **Downstream calls** — search spans, then open the matching trace to confirm the expected services
  were called with the expected outcome.

## Hunt for unexpected behaviour

Within the same window and scope:
- **Errors/exceptions** — search logs for error-level entries / stack traces tied to the correlation
  key or service, even when the API response was 200/OK.
- **Retries & duplicates** — repeated identical operations or duplicate downstream calls in the trace.
- **Latency** — spans far slower than the service's norm.
- **Alerts & incidents** — anything that fired in the window (triggered monitors, paged incidents).

Any of these is a finding even if the response looked correct — record it as a defect or an open
question in the report.

## Capture evidence

For each scenario, save concrete proof: the matching log lines, trace id(s), and any alert name.
Reference them in the QA report so results are verifiable.
