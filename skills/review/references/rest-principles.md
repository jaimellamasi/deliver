# REST principles — review checklist

Caveman rules. Check these when a diff adds or changes REST endpoints.

## Resources

- URL = noun, plural. `/users`, `/orders`. Never verbs (`/getUser`, `/createOrder`).
- Nest to show ownership, max 2 levels deep. `/users/{id}/orders`. Deeper = wrong model.
- Use kebab-case for multi-word. `/line-items`, not `/lineItems`.
- Resource represents thing, not action. Method carries the action.

## HTTP Methods

- GET reads. Never mutates. Safe + idempotent.
- POST creates. Returns 201 + `Location` header. Not idempotent.
- PUT replaces entire resource. Idempotent. Same payload → same result every time.
- PATCH updates partial resource. Not necessarily idempotent unless designed so.
- DELETE removes. Idempotent. Second DELETE = 404 or 204, never 500.
- Never use GET/DELETE with a body. Never use POST when PUT/PATCH is correct.

## Status Codes

- 200 = success with body. 201 = created. 204 = success, no body.
- 400 = client sent garbage. 401 = not authenticated. 403 = authenticated but forbidden. 404 =
  not found. 409 = conflict. 422 = valid syntax, failed validation.
- 500 = server broke. Never return 200 with an error body. Never return 500 for client mistakes.
- 404 on missing resource, not 400. 401 when no/bad token, not 403.

## Request/Response Shape

- Response body is JSON. `Content-Type: application/json` always set.
- Consistent field naming across all endpoints. Pick camelCase or snake_case. Never mix.
- Don't return raw arrays at root. Wrap: `{ "data": [...] }`. Leaves room for metadata.
- Never expose internal IDs, DB primary keys, or stack traces in responses.
- Dates in ISO 8601 (`2024-01-15T10:30:00Z`). Never timestamps, never local formats.

## Idempotency

- GET, PUT, DELETE must be idempotent. Repeat = same outcome.
- POST that creates must not be idempotent by default — guard with idempotency key if
  retry-safe behaviour is needed.
- Idempotency key = client-generated UUID in header (`Idempotency-Key`). Server caches result,
  replays on duplicate.

## Versioning

- Version in URL path. `/v1/users`. Not in header, not in query param.
- Never break existing `/v1` contract — add `/v2` instead.
- New field in response = non-breaking. Removed/renamed field = breaking = new version.

## Security

- Auth on every non-public endpoint. No exceptions.
- Never trust input. Validate type, range, length server-side.
- Never return sensitive fields (passwords, tokens, PII) in any response, even partial.
- Rate limiting must exist. 429 with `Retry-After` when exceeded.

## Errors

- Error shape is consistent across all endpoints. One schema, always.
- Minimum: `{ "error": { "code": "...", "message": "..." } }`. Prefer RFC 9457 Problem Details.
- `message` is human-readable. `code` is machine-readable. Both required.
- Never leak stack traces, SQL errors, or internal paths in error responses.

## Note

These are defaults, not absolutes. If the target repo's own conventions
(`AGENTS.md`/`CLAUDE.md`, neighbouring endpoints) deliberately diverge, follow the repo — flag
the divergence only if it looks accidental or inconsistent within the repo itself.
