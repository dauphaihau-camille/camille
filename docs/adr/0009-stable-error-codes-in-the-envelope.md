# Stable Error Codes In The Error Envelope

## Status

Accepted

## Context

The published API contract (ADR 0008) covers failure responses as well as success shapes. Every failed request returns an error envelope with `statusCode`, `timestamp`, `path`, `error`, `message`, and `code`.

Before `code` existed, clients had to branch on `error` or parse `message`. `error` is a Nest exception class name and changes whenever a controller or mapper throws a different exception or a class is renamed. `message` is human-readable prose that may be reworded or returned as a per-field array. Neither is a stable contract: both change with refactors and neither distinguishes business failures that share an HTTP status.

Clients need one field they can match exhaustively, that survives internal refactoring, and that the contract can version. `code` is that field.

## Decision

Every error envelope carries `code`, a canonical `UPPER_SNAKE_CASE` string. `GlobalExceptionFilter` resolves it through `resolveErrorCode` in `apps/api/api/src/platform/http/api-error-code.ts`, in this order:

1. An explicit canonical payload code. When the exception response object carries a string `code` matching `^[A-Z][A-Z0-9]*(?:_[A-Z0-9]+)*$`, it is used verbatim. Any other payload value is ignored.
2. A domain error code preserved by the HTTP mapper. Every `*-http-error-mapper.ts` builds its exception payload with `toHttpErrorPayload`, which derives the code from the domain error's explicit `code` or, absent one, its class name, canonicalised by `toUpperSnakeCase` (`DocumentNotFoundError` becomes `DOCUMENT_NOT_FOUND`). Because the derived value is canonical, step 1 preserves it unchanged.
3. A status fallback from `HTTP_ERROR_CODE_FALLBACKS`.

The envelope never exposes an internal class name as a code value. An exception class name reaches the envelope only through the domain-error path, where it is converted to a canonical code; a non-canonical payload string is rejected rather than passed through. A status absent from the table falls back to `BAD_REQUEST` for 4xx and `INTERNAL_SERVER_ERROR` otherwise. Status itself resolves first: an `HttpException` reports its own status, an object carrying a 4xx or 5xx `status` or `statusCode` reports that value, and anything else is treated as `INTERNAL_SERVER_ERROR`.

| Status | Fallback code |
| --- | --- |
| 400 Bad Request | `BAD_REQUEST` |
| 401 Unauthorized | `UNAUTHORIZED` |
| 403 Forbidden | `FORBIDDEN` |
| 404 Not Found | `NOT_FOUND` |
| 405 Method Not Allowed | `METHOD_NOT_ALLOWED` |
| 409 Conflict | `CONFLICT` |
| 422 Unprocessable Entity | `UNPROCESSABLE_ENTITY` |
| 429 Too Many Requests | `TOO_MANY_REQUESTS` |
| 500 Internal Server Error | `INTERNAL_SERVER_ERROR` |
| 501 Not Implemented | `NOT_IMPLEMENTED` |
| 503 Service Unavailable | `SERVICE_UNAVAILABLE` |
| 504 Gateway Timeout | `GATEWAY_TIMEOUT` |

`error` remains in the envelope for now. It is useful in logs and while debugging, but it is redundant with `code` and is deprecated: it is scheduled for removal in the next major contract version, `3.0.0`. Until then both fields are documented and populated, and `code` is the field clients branch on.

## Consequences

- Clients branch on `code` and treat `message` and `error` as opaque display or diagnostic text.
- Adding a code is additive: existing clients ignore values they do not know, so a new code takes a minor version.
- Removing or renaming a code is breaking and requires a major version.
- The fallback table bounds the vocabulary: an endpoint without a specific domain code reports a generic status code, so endpoint-specific codes are an ongoing refinement rather than a one-time migration.
- The filter is the single place where codes are resolved, and its unit tests pin the derivation order, so a leaked class name or a malformed payload cannot silently become a code.
- `error` and `code` are temporarily duplicated in the documented envelope. Removing `error` in `3.0.0` is itself a breaking change, which is why it is deferred to a major.

## Alternatives Considered

### Keep only `error`

Rejected. It is a Nest exception class name that changes with refactoring, cannot distinguish failures sharing a status, and gives clients no dependable branching key.

### Expose HTTP status only

Rejected. Status codes conflate distinct business failures such as document-not-found, workspace-not-found, and invitation-not-found, forcing clients to fall back to message parsing.

### Encode codes per domain as separate enums

Rejected. Per-domain enums fragment one envelope field into several overlapping vocabularies, complicate generation and contract comparison, and still require a fallback for errors raised outside any domain. One canonical string vocabulary keeps the envelope serialisable and the breaking check mechanical.
