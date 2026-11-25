# 9. Architecture Decisions

This chapter records the decisions that shape the system. ADR-001 fixes the
serverless Python stack, ADR-002 the obstacle and boundary behavior, and
ADR-003 the mission and result contract shared by both adapters. All three are
accepted and none is superseded.

## ADR-001: Serverless Python Stack on AWS

**Status:** Accepted

**Context:**
The Mars Rover kata simulates a rover on a map with obstacles. It receives
a start position, a heading and a list of commands, and reports the final
position and heading. Usage is occasional, so the system should cost nothing
when idle. The course tooling (pre-commit, black, ruff, bandit) is Python based.

**Decision:**
- Engine: pure Python 3.12 package with no I/O dependencies
- API: Amazon API Gateway (HTTP API) invoking an AWS Lambda function
- CLI: local entry point that reads a mission file
- Storage: none, each mission is sent in a single request
- Infrastructure as Code: AWS SAM
- Testing: pytest

**Rationale:**
- Scales to zero and fits the AWS free tier
- A pure engine keeps the navigation rules in one place and is easy to test
- The CLI and the Lambda share the same engine through thin adapters
- SAM fits the serverless model and the CI/CD module

**Consequences:**
- Lambda cold starts add latency to the first request
- No mission history; storing missions would need a data store and a new ADR
- Input formats must be defined explicitly because the kata does not fix them

## ADR-002: Boundary and Obstacle Stop Behavior

**Status:** Accepted

**Context:**
The rover moves on a rectangular map that contains obstacles. The kata
specifies the `F`, `L` and `R` commands and that a move onto an obstacle is
skipped, but it does not define what happens when a move would leave the map.
Without one rule, the CLI, the API and the tests could disagree about the
result of a mission that reaches an edge or an obstacle.

**Decision:**
- A forward move onto a cell outside the rectangular map is a no-op: the rover
  remains in its current cell (the boundary cell). There is no wrap-around.
- A forward move onto an obstacle cell is a no-op: the rover remains in its
  current cell.
- Both rules apply per command. A blocked `F` is not an error and does not end
  the mission; subsequent commands are still executed.
- `L` and `R` always rotate the rover in place, including when it is next to an
  obstacle or at the map boundary.

**Rationale:**
- Boundary and obstacle cells are treated uniformly as impassable, so the
  engine needs one simple "can the rover enter the next cell?" check.
- Avoids surprising teleportation from wrap-around and preserves the intuitive
  meaning of a bounded rectangular map.
- Keeps the engine deterministic and easy to test per cell; the same behavior
  is shared by the CLI and the Lambda adapter.

**Consequences:**
- Boundary and obstacle cases must be covered by pytest tests in the engine.
- Variants that wrap around or treat a blocked move as a terminal error are
  out of scope; changing them requires a new ADR.
- Validation of the start cell (inside the map, not on an obstacle) and of
  unknown commands is handled by the shared mission model; adapters map the
  resulting error to their transport (Chapter 8).

## ADR-003: Mission and Result JSON Contract

**Status:** Accepted

**Context:**
ADR-001 notes that input formats must be defined explicitly because the kata
does not fix them. The CLI and the HTTPS API must accept the same mission and
return the same result, and tests must be able to reuse mission cases across
both adapters. The orientation of `N`, `E`, `S`, `W` on the rectangular map is
also unspecified by the kata.

**Decision:**
- One mission payload for both adapters: `map` with `width`, `height` and
  `obstacles` as `[x, y]` pairs; `start` with `x`, `y` and `heading`;
  `commands` as a string over `F`, `L`, `R`.
- One result payload: `{"x": <int>, "y": <int>, "heading": "<N|E|S|W>"}`.
- Cells are indexed from `(0, 0)` at the lower-left corner; `x` increases east
  and `y` increases north, so `N` moves to `y + 1`, `E` to `x + 1`, `S` to
  `y - 1` and `W` to `x - 1`.
- Obstacles and the start cell must be inside the map, and the start cell must
  not be an obstacle; invalid missions are rejected as described in Chapter 8.
- No trace or history is returned, only the final cell and heading.

**Rationale:**
- A single shared contract prevents the CLI and the API from diverging and
  lets the same pytest cases drive both adapters.
- Simple objects and coordinate pairs are easy to write by hand and to validate.
- Explicit orientation removes ambiguity about turns and about which cells are
  outside the map (ADR-002).

**Consequences:**
- Clients must produce the exact shape; changing it requires a new ADR and
  updates to both adapters, the engine model and the tests.
- Commands are a compact string rather than an array; adapters bound input size.
- The kata needs no schema version field; adding one later requires a new ADR.
