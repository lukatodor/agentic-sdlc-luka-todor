# Navigation

Domain: **NAV** (Navigation Engine)

## NAV-STORY-001: Execute a command sequence and report the final cell and heading

**Priority:** 🔴 Core
**Architecture:** `architecture/03-system-scope-and-context.md`, section 3.2; `architecture/05-building-block-view.md`, sections 5.2, 5.3 and 5.4; ADR-003

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

**Architecture:** `architecture/07-deployment-view.md`, sections 7.2, 7.4 and 7.5; ADR-001

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

**Architecture:** `architecture/05-building-block-view.md`, section 5.1; `architecture/08-cross-cutting-concepts.md`, section 8.5; ADR-001

Not applicable: missions are self-contained and there is no storage container.

## NAV-INFRA-001.3: Event handling

**Architecture:** `architecture/05-building-block-view.md`, section 5.3

Not applicable: adapters call the engine in-process and synchronously; no events are published.

## NAV-INFRA-001.4: Monitoring of engine failures

**Architecture:** `architecture/08-cross-cutting-concepts.md`, section 8.3

### NAV-INFRA-001.4-S1: Unexpected engine errors are logged

**GIVEN**
* The Lambda function is deployed with CloudWatch Logs enabled

**WHEN**
* The engine raises an unexpected exception while executing a mission

**THEN**
* A log entry with the exception type and the request ID appears in CloudWatch Logs

---

## NAV-BE-001.1: Rotate the rover in place

**Architecture:** `architecture/05-building-block-view.md`, sections 5.2 (Rover State) and 5.4

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

**Architecture:** `architecture/05-building-block-view.md`, section 5.2 (Rover State, Grid Rules); ADR-003

### NAV-BE-001.2-S1: Move east on a free cell

**GIVEN**
* A 5x4 map without obstacles
* A rover state at `(2, 1)` facing `E`

**WHEN**
* The command `F` is applied

**THEN**
* The new state is at `(3, 1)` facing `E`

## NAV-BE-001.3: Execute the command string left to right

**Architecture:** `architecture/05-building-block-view.md`, sections 5.2 (Command Executor) and 5.3

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

**Architecture:** `architecture/05-building-block-view.md`, section 5.1 (CLI Adapter); `architecture/03-system-scope-and-context.md`, section 3.2 (Result output)

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
