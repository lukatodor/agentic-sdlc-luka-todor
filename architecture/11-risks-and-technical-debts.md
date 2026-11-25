# 11. Risks and Technical Debts

Items are ordered by priority. A **risk** is a future event that could harm a
quality goal; a **debt** is a shortcut or deferred work that should be paid
back. Each item names its mitigation and the point at which it must be
revisited.

## 11.1 High Priority

### R1: Different Edge-Rule Expectations (Risk)

The kata fixes obstacle skipping but not boundary behavior, so a reviewer or
client may expect wrap-around or a mission that ends at the first blocked
move.

- **Impact:** Wrong results and conflicting tests between the CLI and API.
- **Mitigation:** ADR-002 defines the no-op behavior for both obstacles and
  boundaries; Chapter 6 gives a worked example; the same pytest cases drive
  both adapters.
- **Revisit:** If the product owner changes the rule, supersede ADR-002 rather
  than patching one adapter.

### D1: Design Not Yet Implemented (Debt)

The repository contains arc42 documentation and tooling, but no engine,
adapters, SAM template or pytest suite yet.

- **Impact:** The architecture cannot be validated by running code until the
  later course modules.
- **Mitigation:** Implement the building blocks from Chapter 5 according to
  Chapters 6-8; treat this as planned work in Modules 4-6, not as hidden debt.
- **Revisit:** At the first implementation review, check that code matches the
  chapter contracts.

## 11.2 Medium Priority

### R2: Public API Without Authentication (Risk)

There is no authentication or authorization (Chapter 8). An exposed endpoint
could receive unwanted traffic.

- **Impact:** Noise, unexpected Lambda cost and misleading test data.
- **Mitigation:** Keep the API within the course environment; ADR-001 does not
  provision a custom domain; require a new ADR before public exposure, with
  throttling and access control.
- **Revisit:** Before sharing the endpoint beyond the course.

### R3: Unbounded Mission Size (Risk)

Map dimensions and command strings are validated for shape but no concrete
size limit is fixed yet.

- **Impact:** A very large mission could increase latency or hit the Lambda
  timeout, weakening Q7.
- **Mitigation:** Adapters bound input size as stated in Chapter 8; define
  exact limits during implementation and cover them in adapter tests.
- **Revisit:** When the mission model and adapters are implemented.

### D2: No Mission History (Debt)

ADR-001 stores nothing. Only adapter logs remain, and they intentionally do
not contain full mission payloads.

- **Impact:** A reported wrong result cannot be reconstructed after the fact.
- **Mitigation:** Keep logs for acceptance/rejection and failures; any future
  store requires a new ADR because it changes the stateless design.
- **Revisit:** If operators need audit or replay capability.

### D3: Quality Gates Not Automated (Debt)

The scenarios in Chapter 10 are specified but not yet enforced by a pipeline.

- **Impact:** Regressions in determinism, engine purity or edge rules could be
  merged unnoticed.
- **Mitigation:** Add pytest, ruff and bandit to the CI/CD pipeline in the
  corresponding course module; keep pre-commit hooks from Chapter 2.
- **Revisit:** When the deployment pipeline is introduced.

## 11.3 Low Priority

### R4: Lambda Cold Start (Risk)

The first request after inactivity waits for a new execution environment
(ADR-001).

- **Impact:** Higher latency for the first mission; Q7 measures warm requests.
- **Mitigation:** Accept the trade-off for occasional usage; do not add
  provisioned concurrency because it would break the idle-cost goal in Q6 and
  need a new ADR.
- **Revisit:** Only if measured first-request latency becomes unacceptable.

### R5: Diagram Includes Depend on Upstream Library (Risk)

The `.puml` files include C4-PlantUML from a remote URL at render time.

- **Impact:** Offline rendering or an upstream change could break the pre-commit
  hook and leave SVGs stale.
- **Mitigation:** Generated SVGs are committed and the hook regenerates them
  for staged `.puml` files; consider pinning or vendoring the C4 library.
- **Revisit:** If diagram rendering becomes unreliable.

### D4: Error Contract Is Not Versioned (Debt)

ADR-003 fixes one result shape and Chapter 8 one error shape, with no schema
version field.

- **Impact:** A future contract change could break existing clients silently.
- **Mitigation:** The kata has no external consumers; any change requires a
  new ADR and updates to both adapters and tests.
- **Revisit:** When real clients appear outside the course.
