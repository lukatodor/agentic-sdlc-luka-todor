# 8. Cross-cutting Concepts

## 8.1 Validation

Validation is split by concern and exists once per concern:

- **Adapters (transport):** parse JSON, check required fields and JSON types,
  and reject malformed payloads before calling the engine. The CLI does this
  for the mission file and the Lambda adapter for the request body.
- **Engine model (domain):** check map dimensions, obstacle cells, start cell,
  heading and commands. A start outside the map, a start on an obstacle, a
  non-positive map size or a command outside `F`, `L`, `R` is invalid.
- **Blocked moves are valid:** an obstacle or boundary only makes an `F`
  command a no-op, never an invalid mission (ADR-002).

Building mission values in one shared model keeps CLI and API validation
consistent; adapters only translate the resulting error into their transport.

## 8.2 Error Handling

| Situation | HTTP API result | CLI result |
| --- | --- | --- |
| Malformed JSON or invalid mission | `400` with `{"error": {"code": "INVALID_MISSION", "message": "..."}}` | Message on stderr, non-zero exit code. |
| Unexpected failure | `500` with a generic message; no stack trace returned | Message on stderr, non-zero exit code. |
| Blocked `F` (obstacle or boundary) | Normal `200` with the unchanged cell | Normal result JSON |

Exceptions raised by the engine carry enough context for the adapter to
produce the message. No partial result is returned for a rejected mission.

## 8.3 Logging and Monitoring

- The navigation engine does not log: it is a pure function of its input and
  has no I/O dependency.
- The Lambda adapter logs mission acceptance or rejection and unexpected
  failures through the standard Python logger, which AWS Lambda sends to the
  CloudWatch log group described in Chapter 7.
- The CLI adapter writes diagnostics to stderr only on failure; stdout carries
  the result JSON so it can be piped.
- Logs contain the reason for rejection and request identifiers, not full
  mission payloads or credentials. No metrics or alarms are needed for a kata.

## 8.4 Security

- Remote access uses HTTPS through API Gateway; the CLI needs no credentials.
- The Lambda execution role has least privilege: CloudWatch Logs only.
- No secrets, tokens or accounts are stored in the repository; detect-secrets
  and bandit run in pre-commit (Chapter 2).
- There is no authentication or authorization in the kata because there are no
  user accounts or private data. Exposing the API publicly beyond the course
  would require a new ADR.

## 8.5 Statelessness and Idempotence

- A mission is self-contained; neither adapter nor engine keeps state between
  calls, so retries and concurrent invocations produce the same result.
- The engine has no side effects: no files, sockets, clocks or random values.
- `L` and `R` always change the heading in place, including at the boundary or
  next to an obstacle; a blocked `F` leaves the state unchanged (ADR-002).

## 8.6 Testing Concepts

- pytest tests target the engine directly with table-driven mission cases,
  especially obstacle, boundary and blocked-then-resumed sequences.
- Adapter tests supply a mission file or an API Gateway event and assert the
  resulting stdout or response; they do not start AWS services.
- The same test cases are reused for CLI and API behavior so the two adapters
  cannot drift apart.
