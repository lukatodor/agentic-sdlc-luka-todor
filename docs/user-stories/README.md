# User Stories: Mars Rover

Stories derived from the architecture in `architecture/` (arc42).
Story IDs follow `{DOMAIN}-STORY-{N}`, `{DOMAIN}-{FE|BE|INFRA}-{N}.{X}` and `{STORY-ID}-S{N}`.

## Domains

| Domain | Bounded context / building block | Architecture |
|--------|----------------------------------|--------------|
| NAV | Navigation Engine: Rover State, Command Executor, Grid Rules | `architecture/05-building-block-view.md`, sections [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) and [5.4](../../architecture/05-building-block-view.md#54-internal-data-model); [ADR-002](../../architecture/09-architecture-decisions.md#adr-002-boundary-and-obstacle-stop-behavior) |
| MISSION | Mission Model: mission and result contract, validation | `architecture/05-building-block-view.md`, section [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine); [ADR-003](../../architecture/09-architecture-decisions.md#adr-003-mission-and-result-json-contract) |
| CLI | CLI Adapter | `architecture/05-building-block-view.md`, section [5.1](../../architecture/05-building-block-view.md#51-level-1-containers); `architecture/03-system-scope-and-context.md`, section [3.2](../../architecture/03-system-scope-and-context.md#32-technical-context) |
| API | HTTP API (API Gateway) and Lambda Adapter | `architecture/05-building-block-view.md`, sections [5.1](../../architecture/05-building-block-view.md#51-level-1-containers) and [5.3](../../architecture/05-building-block-view.md#53-interfaces-between-building-blocks); [ADR-001](../../architecture/09-architecture-decisions.md#adr-001-serverless-python-stack-on-aws) |

## Story Inventory

| # | ID | Story | Domain | Priority | Status | File |
|---|----|-------|--------|----------|--------|------|
| 1 | NAV-STORY-001 | Execute a command sequence and report the final cell and heading | NAV | 🔴 Core | ✅ | [navigation.md](navigation.md) |
| 2 | NAV-STORY-002 | Skip forward moves blocked by obstacles or the map boundary | NAV | 🔴 Core | ✅ | [navigation.md](navigation.md) |
| 3 | API-STORY-001 | Submit a mission over HTTPS and receive the result | API | 🔴 Core | ✅ | [api.md](api.md) |
| 4 | CLI-STORY-001 | Run a mission from a JSON file | CLI | Supporting | ⬜ | |
| 5 | MISSION-STORY-001 | Reject missions with a start cell or obstacles outside the map or a start cell on an obstacle | MISSION | Supporting | ✅ | [mission.md](mission.md) |
| 6 | MISSION-STORY-002 | Reject unknown commands and headings | MISSION | Supporting | ⬜ | |
| 7 | API-STORY-002 | Return a clear HTTP error for invalid requests | API | Supporting | ⬜ | |
| 8 | CLI-STORY-002 | Report invalid mission files with an error message and a non-zero exit code | CLI | Supporting | ⬜ | |
| 9 | API-STORY-003 | Monitor API errors and latency | API | Supporting | ⬜ | |

## Pareto Analysis

The three core stories carry most of the value:

- **NAV-STORY-001** is the purpose of the system: without it no mission produces a result.
- **NAV-STORY-002** implements [ADR-002](../../architecture/09-architecture-decisions.md#adr-002-boundary-and-obstacle-stop-behavior), the rule that makes results on a map with obstacles and edges correct.
- **API-STORY-001** exercises every container (HTTP API, Lambda Adapter, Navigation Engine) and is the deployed entry point.

All other stories add input validation, an alternative channel or operations support on top of the core.

## Pareto Progress

📊 Pareto Progress: 3/3 core stories (100% of 20% core stories)
🎯 Core functionality coverage: ~80% of 80% target
