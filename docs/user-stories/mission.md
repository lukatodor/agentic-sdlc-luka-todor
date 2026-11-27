# Mission

Domain: **MISSION** (Mission Model: mission and result contract, validation)

## MISSION-STORY-001: Reject missions with a start cell or obstacles outside the map or a start cell on an obstacle

**Priority:** Supporting
**Architecture:** `architecture/05-building-block-view.md`, section [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) (Mission Model); `architecture/08-cross-cutting-concepts.md`, sections [8.1](../../architecture/08-cross-cutting-concepts.md#81-validation) and [8.2](../../architecture/08-cross-cutting-concepts.md#82-error-handling); `architecture/03-system-scope-and-context.md`, section [3.2](../../architecture/03-system-scope-and-context.md#32-technical-context); [ADR-003](../../architecture/09-architecture-decisions.md#adr-003-mission-and-result-json-contract)

AS A mission operator
I WANT invalid mission coordinates to be rejected before execution
SO THAT I receive a clear error instead of an incorrect rover result

### MISSION-STORY-001-S1: Start cell outside the map is rejected

**GIVEN**
* A mission with map `width` 5, `height` 4, obstacles `[]`, start `x` 5, `y` 1, heading `N`, and commands `F`

**WHEN**
* The mission is validated

**THEN**
* The mission is rejected with error code `INVALID_MISSION`
* The error message is `start cell (5, 1) is outside the 5x4 map`
* No engine execution occurs

### MISSION-STORY-001-S2: Start cell on an obstacle is rejected

**GIVEN**
* A mission with map `width` 5, `height` 4, obstacles `[[1, 1]]`, start `x` 1, `y` 1, heading `N`, and commands `F`

**WHEN**
* The mission is validated

**THEN**
* The mission is rejected with error code `INVALID_MISSION`
* The error message is `start cell (1, 1) is on an obstacle`
* No engine execution occurs

---

## MISSION-INFRA-001.1: Deployment

**Architecture:** `architecture/07-deployment-view.md`, sections [7.2](../../architecture/07-deployment-view.md#72-aws-environment) and [7.4](../../architecture/07-deployment-view.md#74-building-block-placement); [ADR-001](../../architecture/09-architecture-decisions.md#adr-001-serverless-python-stack-on-aws)

Not applicable: mission validation is part of the shared engine/model package bundled by NAV-INFRA-001.1 and API-INFRA-001.1; no separate deployment artifact is required.

## MISSION-INFRA-001.2: Data store

**Architecture:** `architecture/05-building-block-view.md`, section [5.1](../../architecture/05-building-block-view.md#51-level-1-containers); `architecture/08-cross-cutting-concepts.md`, section [8.5](../../architecture/08-cross-cutting-concepts.md#85-statelessness-and-idempotence); [ADR-001](../../architecture/09-architecture-decisions.md#adr-001-serverless-python-stack-on-aws)

Not applicable: missions are self-contained and there is no storage container.

## MISSION-INFRA-001.3: Event handling

**Architecture:** `architecture/05-building-block-view.md`, section [5.3](../../architecture/05-building-block-view.md#53-interfaces-between-building-blocks)

Not applicable: validation is performed synchronously in-process before engine execution; no events are published.

## MISSION-INFRA-001.4: Monitoring and observability

**Architecture:** `architecture/08-cross-cutting-concepts.md`, section [8.3](../../architecture/08-cross-cutting-concepts.md#83-logging-and-monitoring)

Not applicable: validation rejection logging is covered by adapter monitoring (NAV-INFRA-001.4 for unexpected failures; API-INFRA-001.4/API-STORY-003 for API request handling). The engine model itself performs no I/O or logging.

---

## MISSION-BE-001.1: Reject a start cell outside the map

**Architecture:** `architecture/05-building-block-view.md`, section [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) (Mission Model); `architecture/08-cross-cutting-concepts.md`, section [8.1](../../architecture/08-cross-cutting-concepts.md#81-validation); [ADR-003](../../architecture/09-architecture-decisions.md#adr-003-mission-and-result-json-contract)

### MISSION-BE-001.1-S1: Start x equals map width is outside the map

**GIVEN**
* A mission value with map width 5, height 4, obstacles `[]`, start `(5, 1)` heading `N`, and commands `F`

**WHEN**
* The mission model validates the mission

**THEN**
* Validation fails with error code `INVALID_MISSION`
* The error message is `start cell (5, 1) is outside the 5x4 map`

## MISSION-BE-001.2: Reject an obstacle outside the map

**Architecture:** `architecture/05-building-block-view.md`, section [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) (Mission Model); `architecture/08-cross-cutting-concepts.md`, section [8.1](../../architecture/08-cross-cutting-concepts.md#81-validation); [ADR-003](../../architecture/09-architecture-decisions.md#adr-003-mission-and-result-json-contract)

### MISSION-BE-001.2-S1: Obstacle with negative coordinate is rejected

**GIVEN**
* A mission value with map width 5, height 4, obstacles `[(-1, 2)]`, start `(1, 1)` heading `N`, and commands `F`

**WHEN**
* The mission model validates the mission

**THEN**
* Validation fails with error code `INVALID_MISSION`
* The error message is `obstacle (-1, 2) is outside the 5x4 map`

## MISSION-BE-001.3: Reject a start cell on an obstacle

**Architecture:** `architecture/05-building-block-view.md`, section [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) (Mission Model); `architecture/08-cross-cutting-concepts.md`, section [8.1](../../architecture/08-cross-cutting-concepts.md#81-validation); [ADR-003](../../architecture/09-architecture-decisions.md#adr-003-mission-and-result-json-contract)

### MISSION-BE-001.3-S1: Start cell coincides with an obstacle

**GIVEN**
* A mission value with map width 5, height 4, obstacles `[(1, 1)]`, start `(1, 1)` heading `N`, and commands `F`

**WHEN**
* The mission model validates the mission

**THEN**
* Validation fails with error code `INVALID_MISSION`
* The error message is `start cell (1, 1) is on an obstacle`

---

## MISSION-FE-001.1: Report invalid mission coordinates in the CLI

**Architecture:** `architecture/05-building-block-view.md`, section [5.1](../../architecture/05-building-block-view.md#51-level-1-containers) (CLI Adapter); `architecture/08-cross-cutting-concepts.md`, section [8.2](../../architecture/08-cross-cutting-concepts.md#82-error-handling)

The CLI is the user-facing interface for mission validation feedback in local runs.

### MISSION-FE-001.1-S1: CLI rejects a mission whose start cell is outside the map

**GIVEN**
* A mission file with map `width` 5, `height` 4, obstacles `[]`, start `x` 5, `y` 1, heading `N`, and commands `F`

**WHEN**
* The operator runs the CLI with the path to that file

**THEN**
* stderr contains `INVALID_MISSION: start cell (5, 1) is outside the 5x4 map`
* The exit code is non-zero
* stdout is empty

---

## Implementation Order

1. No new INFRA (covered by NAV-INFRA-001.1, API-INFRA-001.1, and adapter monitoring)
2. MISSION-BE-001.1, MISSION-BE-001.2, MISSION-BE-001.3
3. MISSION-FE-001.1
4. MISSION-STORY-001 end to end (S1, S2)
