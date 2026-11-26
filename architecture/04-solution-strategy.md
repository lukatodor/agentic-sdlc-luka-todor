# 4. Solution Strategy

The system is built around a pure navigation engine wrapped by thin delivery
adapters. A mission enters as a self-contained value, the engine folds the
command sequence over the rover state, and the resulting cell and heading are
returned to the caller. No state survives a mission.

## 4.1 Technology Decisions

- **Engine:** pure Python 3.12, standard library only, with no file, network or
  AWS dependencies (ADR-001).
- **Remote delivery:** Amazon API Gateway HTTP API invoking a Python AWS Lambda
  function; one mission per synchronous request (ADR-001).
- **Local delivery:** a CLI adapter that reads a mission JSON file and prints
  the result JSON (ADR-001).
- **Infrastructure:** one AWS SAM template for API, function and permissions.
- **Storage:** none; no database, cache, queue or message broker.
- **Testing:** pytest against the engine and the adapters (ADR-001).

## 4.2 Top-Level Approach and Patterns

- **Ports and adapters (hexagonal):** the engine owns the domain rules; CLI and
  Lambda adapters translate their transport (file/stdout, HTTP JSON) into
  domain values and back. Both adapters call the same engine function, so the
  rules exist in exactly one place.
- **Domain language:** mission, map, obstacle, cell, rover, heading and command
  are named after the kata rather than after transport or AWS concepts.
- **Immutable state:** a rover state (cell plus heading) is replaced for each
  command; turns rotate in place and a forward move produces a new cell only
  when the target is enterable.
- **One movement guard:** obstacle cells and cells outside the map are both
  impassable. A blocked `F` leaves the state unchanged but does not abort the
  command sequence, implementing ADR-002 uniformly.
- **Stateless execution:** no globals or per-user state, so repeated and
  concurrent Lambda invocations behave identically.
- **Explicit contract:** the JSON mission and result shapes and the coordinate
  convention are fixed in Chapter 3, because the kata does not define them.

## 4.3 Satisfying the Quality Goals

| Quality goal (Chapter 1) | Strategy |
| --- | --- |
| Correctness and determinism | Pure function from mission to result; a single enterable-cell guard covers obstacle and boundary cases; ADR-002 fixes the edge semantics. |
| Testability | pytest imports the engine directly; table-driven cases cover turns, obstacles, boundaries and blocked-then-resumed commands without AWS. |
| Modularity and portability | Standard-library engine, thin adapters, no duplicated rules; the CLI works locally and the Lambda uses the same code. |
| Simplicity and low operating cost | KISS/YAGNI: smallest viable architecture, no database, serverless scale to zero, one SAM template. |
| Changeability | A new delivery channel adds an adapter; a rule change stays in the engine; ADRs record choices that are expensive to reverse. |

## 4.4 Decision Boundaries

- ADR-001 fixes the serverless Python stack and the thin-adapter split;
  ADR-002 fixes obstacle and boundary behavior. Neither is superseded.
- No event-driven communication is introduced: API Gateway invokes Lambda
  synchronously, as specified in ADR-001.
- Adapter concerns (JSON validation, error responses and logging) are detailed
  in Chapter 8; the static decomposition follows in Chapter 5, runtime
  interaction in Chapter 6 and AWS placement in Chapter 7.
