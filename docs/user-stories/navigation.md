# Navigation

Domain: **NAV** (Navigation Engine)

## NAV-STORY-001: Execute a command sequence and report the final cell and heading

**Priority:** 🔴 Core
**Architecture:** `architecture/03-system-scope-and-context.md`, section [3.2](../../architecture/03-system-scope-and-context.md#32-technical-context); `architecture/05-building-block-view.md`, sections [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine), [5.3](../../architecture/05-building-block-view.md#53-interfaces-between-building-blocks) and [5.4](../../architecture/05-building-block-view.md#54-internal-data-model); [ADR-003](../../architecture/09-architecture-decisions.md#adr-003-mission-and-result-json-contract)

AS A mission operator
I WANT to send a map, a start cell with a heading and a command string to the rover
SO THAT I know in which cell the rover ends and which way it faces

### NAV-STORY-001-S1: Move forward, turn right and move again

**GIVEN**
* A map with `width` 5, `height` 4 and no obstacles
* The rover starts at `x` 1, `y` 1 facing `N`

**WHEN**
* The mission is executed with the commands `FFRFF`

**THEN**
* The result is `{"x": 3, "y": 3, "heading": "E"}`

### NAV-STORY-001-S2: Turn in place only

**GIVEN**
* A map with `width` 5, `height` 4 and no obstacles
* The rover starts at `x` 2, `y` 1 facing `N`

**WHEN**
* The mission is executed with the commands `RRR`

**THEN**
* The result is `{"x": 2, "y": 1, "heading": "W"}`

---

## NAV-INFRA-001.1: Package the Navigation Engine with the Lambda function

**Architecture:** `architecture/07-deployment-view.md`, sections [7.2](../../architecture/07-deployment-view.md#72-aws-environment), [7.4](../../architecture/07-deployment-view.md#74-building-block-placement) and [7.5](../../architecture/07-deployment-view.md#75-deployment-and-operations); [ADR-001](../../architecture/09-architecture-decisions.md#adr-001-serverless-python-stack-on-aws)

The engine is a Python 3.12 package bundled into the Lambda deployment artifact defined in the AWS SAM template.

### NAV-INFRA-001.1-S1: Engine is importable in the deployed function

**GIVEN**
* The SAM template references the Lambda Adapter with the engine package as a dependency

**WHEN**
* `sam build` and `sam deploy` run

**THEN**
* The stack deploys without errors
* The Lambda function imports the engine without an `ImportError`

## NAV-INFRA-001.2: Data store

**Architecture:** `architecture/05-building-block-view.md`, section [5.1](../../architecture/05-building-block-view.md#51-level-1-containers); `architecture/08-cross-cutting-concepts.md`, section [8.5](../../architecture/08-cross-cutting-concepts.md#85-statelessness-and-idempotence); [ADR-001](../../architecture/09-architecture-decisions.md#adr-001-serverless-python-stack-on-aws)

Not applicable: missions are self-contained and there is no storage container.

## NAV-INFRA-001.3: Event handling

**Architecture:** `architecture/05-building-block-view.md`, section [5.3](../../architecture/05-building-block-view.md#53-interfaces-between-building-blocks)

Not applicable: adapters call the engine in-process and synchronously; no events are published.

## NAV-INFRA-001.4: Monitoring of engine failures

**Architecture:** `architecture/08-cross-cutting-concepts.md`, section [8.3](../../architecture/08-cross-cutting-concepts.md#83-logging-and-monitoring)

### NAV-INFRA-001.4-S1: Unexpected engine errors are logged

**GIVEN**
* The Lambda function is deployed with CloudWatch Logs enabled

**WHEN**
* The engine raises an unexpected exception while executing a mission

**THEN**
* A log entry with the exception type and the request ID appears in CloudWatch Logs

---

## NAV-BE-001.1: Rotate the rover in place

**Architecture:** `architecture/05-building-block-view.md`, sections [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) (Rover State) and [5.4](../../architecture/05-building-block-view.md#54-internal-data-model)

### NAV-BE-001.1-S1: Turn right from north

**GIVEN**
* A rover state at `(2, 1)` facing `N`

**WHEN**
* The command `R` is applied

**THEN**
* The new state is at `(2, 1)` facing `E`

### NAV-BE-001.1-S2: Turn left from north

**GIVEN**
* A rover state at `(2, 1)` facing `N`

**WHEN**
* The command `L` is applied

**THEN**
* The new state is at `(2, 1)` facing `W`

## NAV-BE-001.2: Move the rover forward according to its heading

**Architecture:** `architecture/05-building-block-view.md`, section [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) (Rover State, Grid Rules); [ADR-003](../../architecture/09-architecture-decisions.md#adr-003-mission-and-result-json-contract)

### NAV-BE-001.2-S1: Move east on a free cell

**GIVEN**
* A 5x4 map without obstacles
* A rover state at `(2, 1)` facing `E`

**WHEN**
* The command `F` is applied

**THEN**
* The new state is at `(3, 1)` facing `E`

## NAV-BE-001.3: Execute the command string left to right

**Architecture:** `architecture/05-building-block-view.md`, sections [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) (Command Executor) and [5.3](../../architecture/05-building-block-view.md#53-interfaces-between-building-blocks)

### NAV-BE-001.3-S1: Engine returns the final state

**GIVEN**
* A mission value with a 5x4 map, start `(1, 1)` facing `N` and commands `FFRFF`

**WHEN**
* `execute(mission)` is called

**THEN**
* The result value is cell `(3, 3)` with heading `E`
* The mission value is unchanged

---

## NAV-FE-001.1: Show the final position to the operator

**Architecture:** `architecture/05-building-block-view.md`, section [5.1](../../architecture/05-building-block-view.md#51-level-1-containers) (CLI Adapter); `architecture/03-system-scope-and-context.md`, section [3.2](../../architecture/03-system-scope-and-context.md#32-technical-context) (Result output)

The CLI is the user-facing interface of this system.

### NAV-FE-001.1-S1: Print the result JSON

**GIVEN**
* A mission file with a 5x4 map, no obstacles, start `(1, 1)` facing `N` and commands `FFRFF`

**WHEN**
* The operator runs the CLI with the path to that file

**THEN**
* stdout contains exactly `{"x": 3, "y": 3, "heading": "E"}`
* The exit code is 0

---

## Implementation Order

1. NAV-INFRA-001.1, NAV-INFRA-001.4
2. NAV-BE-001.1, NAV-BE-001.2, NAV-BE-001.3
3. NAV-FE-001.1
4. NAV-STORY-001 end to end (S1, S2)

---

## NAV-STORY-002: Skip forward moves blocked by obstacles or the map boundary

**Priority:** 🔴 Core
**Architecture:** `architecture/05-building-block-view.md`, sections [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) (Command Executor, Grid Rules) and [5.4](../../architecture/05-building-block-view.md#54-internal-data-model); `architecture/06-runtime-view.md`, section [6.3](../../architecture/06-runtime-view.md#63-blocked-move-example); `architecture/08-cross-cutting-concepts.md`, section [8.5](../../architecture/08-cross-cutting-concepts.md#85-statelessness-and-idempotence); `architecture/10-quality-requirements.md`, section [10.2](../../architecture/10-quality-requirements.md#102-quality-scenarios) (Q2); [ADR-002](../../architecture/09-architecture-decisions.md#adr-002-boundary-and-obstacle-stop-behavior)

AS A mission operator
I WANT the rover to skip a forward move when an obstacle or the map boundary blocks it
SO THAT I still get the final cell and heading after the remaining commands

### NAV-STORY-002-S1: Obstacle blocks forward moves and the mission continues

**GIVEN**
* A map with `width` 5, `height` 4 and an obstacle at `(1, 2)`
* The rover starts at `x` 1, `y` 1 facing `N`

**WHEN**
* The mission is executed with the commands `FFR`

**THEN**
* The result is `{"x": 1, "y": 1, "heading": "E"}`

### NAV-STORY-002-S2: Map boundary blocks forward moves and the mission continues

**GIVEN**
* A map with `width` 5, `height` 4 and no obstacles
* The rover starts at `x` 4, `y` 3 facing `N`

**WHEN**
* The mission is executed with the commands `FFR`

**THEN**
* The result is `{"x": 4, "y": 3, "heading": "E"}`

### NAV-STORY-002-S3: Blocked forward, turn, then forward moves

**GIVEN**
* A map with `width` 5, `height` 4 and an obstacle at `(1, 2)`
* The rover starts at `x` 1, `y` 1 facing `N`

**WHEN**
* The mission is executed with the commands `FRF`

**THEN**
* The result is `{"x": 2, "y": 1, "heading": "E"}`

---

## NAV-INFRA-002.1: Package the Navigation Engine with the Lambda function

**Architecture:** `architecture/07-deployment-view.md`, sections [7.2](../../architecture/07-deployment-view.md#72-aws-environment), [7.4](../../architecture/07-deployment-view.md#74-building-block-placement) and [7.5](../../architecture/07-deployment-view.md#75-deployment-and-operations); [ADR-001](../../architecture/09-architecture-decisions.md#adr-001-serverless-python-stack-on-aws)

Not applicable: covered by NAV-INFRA-001.1; blocked-move behavior adds no deployment artifact.

## NAV-INFRA-002.2: Data store

**Architecture:** `architecture/05-building-block-view.md`, section [5.1](../../architecture/05-building-block-view.md#51-level-1-containers); `architecture/08-cross-cutting-concepts.md`, section [8.5](../../architecture/08-cross-cutting-concepts.md#85-statelessness-and-idempotence); [ADR-001](../../architecture/09-architecture-decisions.md#adr-001-serverless-python-stack-on-aws)

Not applicable: missions are self-contained and there is no storage container.

## NAV-INFRA-002.3: Event handling

**Architecture:** `architecture/05-building-block-view.md`, section [5.3](../../architecture/05-building-block-view.md#53-interfaces-between-building-blocks)

Not applicable: adapters call the engine in-process and synchronously; no events are published.

## NAV-INFRA-002.4: Monitoring of engine failures

**Architecture:** `architecture/08-cross-cutting-concepts.md`, section [8.3](../../architecture/08-cross-cutting-concepts.md#83-logging-and-monitoring)

Not applicable: covered by NAV-INFRA-001.4; blocked moves are normal no-op behavior, not failures.

---

## NAV-BE-002.1: Treat an obstacle cell as impassable

**Architecture:** `architecture/05-building-block-view.md`, section [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) (Grid Rules); [ADR-002](../../architecture/09-architecture-decisions.md#adr-002-boundary-and-obstacle-stop-behavior)

### NAV-BE-002.1-S1: Forward move into an obstacle is a no-op

**GIVEN**
* A 5x4 map with an obstacle at `(1, 2)`
* A rover state at `(1, 1)` facing `N`

**WHEN**
* The command `F` is applied

**THEN**
* The new state is at `(1, 1)` facing `N`

## NAV-BE-002.2: Treat a cell outside the map as impassable

**Architecture:** `architecture/05-building-block-view.md`, section [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) (Grid Rules); [ADR-002](../../architecture/09-architecture-decisions.md#adr-002-boundary-and-obstacle-stop-behavior)

### NAV-BE-002.2-S1: Forward move past the north boundary is a no-op

**GIVEN**
* A 5x4 map without obstacles
* A rover state at `(4, 3)` facing `N`

**WHEN**
* The command `F` is applied

**THEN**
* The new state is at `(4, 3)` facing `N`

## NAV-BE-002.3: Continue executing commands after a blocked forward move

**Architecture:** `architecture/05-building-block-view.md`, section [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) (Command Executor); `architecture/06-runtime-view.md`, section [6.3](../../architecture/06-runtime-view.md#63-blocked-move-example); [ADR-002](../../architecture/09-architecture-decisions.md#adr-002-boundary-and-obstacle-stop-behavior)

### NAV-BE-002.3-S1: Commands after a blocked move are applied

**GIVEN**
* A mission value with a 5x4 map, an obstacle at `(1, 2)`, start `(1, 1)` facing `N` and commands `FFR`

**WHEN**
* `execute(mission)` is called

**THEN**
* The result value is cell `(1, 1)` with heading `E`
* The mission value is unchanged

### NAV-BE-002.3-S2: Blocked forward, turn, then forward moves

**GIVEN**
* A mission value with a 5x4 map, an obstacle at `(1, 2)`, start `(1, 1)` facing `N` and commands `FRF`

**WHEN**
* `execute(mission)` is called

**THEN**
* The result value is cell `(2, 1)` with heading `E`
* The mission value is unchanged

---

## NAV-FE-002.1: Show the final position after blocked moves

**Architecture:** `architecture/05-building-block-view.md`, section [5.1](../../architecture/05-building-block-view.md#51-level-1-containers) (CLI Adapter); `architecture/03-system-scope-and-context.md`, section [3.2](../../architecture/03-system-scope-and-context.md#32-technical-context) (Result output)

### NAV-FE-002.1-S1: Print the result JSON for a mission with blocked moves

**GIVEN**
* A mission file with a 5x4 map, an obstacle at `(1, 2)`, start `(1, 1)` facing `N` and commands `FFR`

**WHEN**
* The operator runs the CLI with the path to that file

**THEN**
* stdout contains exactly `{"x": 1, "y": 1, "heading": "E"}`
* The exit code is 0

---

## Implementation Order

1. No new INFRA (covered by NAV-INFRA-001.1 and NAV-INFRA-001.4)
2. NAV-BE-002.1, NAV-BE-002.2, NAV-BE-002.3
3. NAV-FE-002.1
4. NAV-STORY-002 end to end (S1, S2, S3)
