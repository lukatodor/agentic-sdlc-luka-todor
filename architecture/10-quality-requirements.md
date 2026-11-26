# 10. Quality Requirements

## 10.1 Quality Tree

The goals below refine the priorities set in Chapter 1. Higher branches are
the goals; leaves are the measurable aspects used in the scenarios.

- **Correctness and determinism** (priority 1)
  - Navigation rules for `F`, `L`, `R` applied in order
  - Obstacle and boundary no-ops without wrap-around (ADR-002)
  - Same mission yields the same result every time
  - Invalid missions rejected instead of silently mis-executed
- **Testability** (priority 2)
  - Engine testable by direct pytest import
  - Shared cases for CLI and API adapters
  - No AWS services needed for unit tests
- **Modularity and portability** (priority 3)
  - Engine free of AWS and I/O dependencies
  - One engine implementation used by both adapters
  - Domain language in names
- **Simplicity and low operating cost** (priority 4)
  - No database or always-on infrastructure
  - Scale to zero between missions
  - Small deployment surface (one SAM stack)
- **Changeability** (priority 5)
  - Rule change confined to the engine
  - New delivery channel adds an adapter only
  - Decisions recorded as ADRs

## 10.2 Quality Scenarios

### Q1: Deterministic Result

- **Source:** Mission Operator.
- **Stimulus:** The same mission JSON is submitted 100 times, alternating
  between the CLI and the HTTPS API.
- **Artifact:** Navigation engine and both adapters.
- **Environment:** Local Python 3.12 process and the deployed Lambda function.
- **Response:** Every call returns the same final cell and heading.
- **Measure:** 100/100 results identical; no result depends on earlier calls.

### Q2: Blocked Move Continues the Mission

- **Source:** Mission Operator or pytest case.
- **Stimulus:** A command `F` targets an obstacle or a cell outside the map,
  followed by more commands.
- **Artifact:** Command Executor and Grid Rules.
- **Environment:** Engine test suite, CLI run or API request.
- **Response:** The blocked `F` leaves the cell unchanged; later commands are
  still applied.
- **Measure:** All obstacle and boundary cases from ADR-002 pass, including the
  `FFR` example in Chapter 6; zero wrap-around moves.

### Q3: Invalid Mission Rejected

- **Source:** Mission Operator.
- **Stimulus:** A mission has malformed JSON, an unknown command, a start cell
  outside the map or a start cell on an obstacle.
- **Artifact:** Adapter transport validation and shared mission model.
- **Environment:** CLI invocation or API request.
- **Response:** A clear rejection with the error shape from Chapter 8; the
  engine returns no partial result.
- **Measure:** 100% of defined invalid-input cases produce the documented
  error, a non-zero CLI exit or HTTP `400`, and no engine execution.

### Q4: Tests Need No Cloud

- **Source:** Developer.
- **Stimulus:** The developer runs `pytest` on a fresh checkout without AWS
  credentials, network access or a deployed stack.
- **Artifact:** pytest suite.
- **Environment:** Local Python 3.12 virtual environment.
- **Response:** All tests pass.
- **Measure:** Zero tests require AWS or a running server; engine cases cover
  every command and every edge rule in ADR-002; the suite completes in under
  five seconds on a developer laptop.

### Q5: Engine Has No I/O Dependency

- **Source:** Reviewer or automated check.
- **Stimulus:** The engine package is inspected for imports and file, socket or
  AWS calls.
- **Artifact:** Navigation Engine package.
- **Environment:** Source tree, before deployment.
- **Response:** Only standard-library domain code is present.
- **Measure:** Zero imports of `boto3`, `requests`, `urllib`, `socket` or
  `pathlib` in the engine package, and exactly one implementation of the
  command-application logic shared by both adapters.

### Q6: Idle Cost Is Zero

- **Source:** DevOps / deployer.
- **Stimulus:** No mission is submitted for an extended period.
- **Artifact:** AWS SAM stack.
- **Environment:** Deployed AWS account.
- **Response:** No compute or storage resource remains running or billed
  outside the free tier.
- **Measure:** Zero always-on resources in the template; Lambda scales to zero
  and the stack contains no database, cache or provisioned concurrency.

### Q7: Short Mission Response Time

- **Source:** Mission Operator.
- **Stimulus:** A mission with at most 100 commands on a `100 x 100` map is sent
  to the API after the function is warm.
- **Artifact:** Lambda adapter and engine.
- **Environment:** Deployed API with a warm execution environment.
- **Response:** The result JSON is returned.
- **Measure:** End-to-end response under 500 ms excluding a possible cold
  start (ADR-001).
