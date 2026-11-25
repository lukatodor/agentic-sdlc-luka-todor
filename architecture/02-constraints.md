# 2. Architecture Constraints

## 2.1 Technical Constraints

| Constraint | Source | Consequence for the design |
| --- | --- | --- |
| Python 3.12 | `pyproject.toml` (`requires-python >=3.12`, black and ruff target `py312`) | The engine and both adapters are Python; no second runtime language. |
| Pure navigation engine | ADR-001, architecture principles | The engine depends only on the standard library; it performs no file, network or AWS I/O. |
| AWS HTTP API plus Lambda | ADR-001 | Each mission is one stateless request/response; input and output must be JSON-serializable. |
| AWS SAM | ADR-001 | API Gateway, Lambda, permissions and deployment are described in one infrastructure template. |
| No database | ADR-001 | The mission (map, start, heading, commands) travels with the request or CLI file; no persistence layer. |
| CLI adapter | ADR-001 | Local entry point reads a mission file and calls the same engine as Lambda. |
| pytest | ADR-001 | Navigation rules are testable by importing the engine, without AWS or a running server. |
| Pre-commit toolchain | `.pre-commit-config.yaml` | black, isort, ruff, bandit, detect-secrets, PlantUML validation and commit-message checks gate every change. |

## 2.2 Organizational Constraints

- The project is Module 2 of the Agentic SDLC course; the architecture is
  documented with arc42 and reviewed one chapter at a time. Work stops for
  approval after each chapter before the next one is generated.
- Existing artifacts are preserved and extended: the C4 context diagram in
  `architecture/diagrams/c4-context.puml`, ADR-001 and ADR-002 in
  `architecture/09-architecture-decisions.md`.
- ADRs are numbered sequentially and never deleted; a changed decision is
  superseded by a new ADR (ADR template in the architecture agent rules).
- Commits follow `#<issue> <type>(<scope>): <description>`, enforced by
  `scripts/hooks/check-commit-msg.sh`; course progress is tracked in the
  README checklist.
- Scope and time are limited to a kata. Occasional usage and the AWS free tier
  keep the target operating cost at zero when idle, so stateful or always-on
  infrastructure is excluded.

## 2.3 Conventions

- **Documentation:** arc42 chapter files in `architecture/` named
  `01-introduction-and-goals.md`, `02-constraints.md`, and so on; diagrams in
  `architecture/diagrams/` as `.puml`.
- **Diagrams:** C4 model with the C4-PlantUML standard library, first line
  `@startuml <filename-without-extension>`, rendered to SVG with
  `java -jar ~/.local/share/plantuml/plantuml.jar -tsvg <file>` and validated by
  `scripts/hooks/validate-plantuml.sh`.
- **Decisions:** ADRs use Status, Context, Decision, Rationale and
  Consequences, in that order.
- **Design style:** Domain-Driven Design names (mission, rover, obstacle,
  heading, command); ports and adapters keep AWS and I/O out of the domain
  engine; KISS and YAGNI favor the smallest architecture that meets the
  quality goals from Chapter 1.
- **Code style:** black with line length 88, isort with the black profile and
  ruff rules `E`, `F`, `W`, `I`, `N`, `UP`, `B`, `SIM`; bandit and
  detect-secrets must pass, and no credentials are stored in the repository.
