# 9. Architecture Decisions

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
