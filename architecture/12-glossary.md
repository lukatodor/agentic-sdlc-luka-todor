# 12. Glossary

## Domain Terms

| Term | Definition |
| --- | --- |
| **Blocked move** | An `F` command whose target cell is an obstacle or outside the map; it is a no-op, not an error (ADR-002). |
| **Boundary** | The edge of the rectangular map. A forward move past it is blocked; there is no wrap-around. |
| **Cell** | One coordinate `(x, y)` on the map, indexed from `(0, 0)` at the lower-left corner; `x` increases east and `y` increases north. |
| **Command** | One character in a mission: `F` (forward one cell), `L` (turn left 90 degrees) or `R` (turn right 90 degrees). |
| **Heading** | Rover direction `N`, `E`, `S` or `W`. `N` moves to `y + 1`, `E` to `x + 1`, `S` to `y - 1`, `W` to `x - 1`. |
| **Map** | Rectangular grid described by `width`, `height` and a set of obstacle cells. |
| **Mission** | Self-contained input value: map, start cell, initial heading and command string. No state is kept between missions. |
| **Mission File** | JSON file holding one mission, read by the CLI adapter. |
| **Obstacle** | An impassable cell on the map. |
| **Result** | Final cell and heading returned after all commands are applied; no trace or history. |
| **Rover** | The simulated vehicle whose state is a cell plus a heading. |
| **Rover State** | Immutable value replaced by each command; an unchanged state represents a blocked move or a turn-in-place as appropriate. |
| **Wrap-around** | Leaving the map and reappearing on the opposite edge; explicitly not supported. |

## Building Block Terms

| Term | Definition |
| --- | --- |
| **CLI Adapter** | Local Python entry point that reads a mission file, calls the engine and prints the result JSON. |
| **Command Executor** | Engine component that applies commands in order and returns the final rover state. |
| **Engine model** | Shared value objects for mission data; validates domain constraints for both adapters. |
| **Grid Rules** | Engine component that decides whether the next cell is inside the map and obstacle-free. |
| **Lambda Adapter** | AWS Lambda function that parses an HTTP request, calls the engine and formats the response. |
| **Mission Model** | Engine component holding map dimensions, obstacles, start cell, heading and commands. |
| **Navigation Engine** | Pure Python 3.12 package containing all navigation rules; no I/O or AWS dependencies. |
| **Ports and adapters** | Design style in which the engine exposes an in-process API and outer adapters translate files and HTTP into domain values. |

## Technical and Documentation Terms

| Term | Definition |
| --- | --- |
| **ADR** | Architecture Decision Record: Status, Context, Decision, Rationale and Consequences for an expensive-to-change choice. |
| **API Gateway** | AWS managed service exposing the HTTPS endpoint and forwarding requests to Lambda. |
| **arc42** | Documentation template used for the twelve architecture chapters in `architecture/`. |
| **AWS SAM** | Infrastructure-as-code tool used to define and deploy the API, function and permissions in one stack. |
| **C4 model** | Diagram notation for Context, Container, Component and Deployment views; sequence diagrams show runtime interaction. |
| **CloudWatch Logs** | Managed AWS log group receiving Lambda adapter diagnostics. |
| **PlantUML** | Tool that renders the `.puml` diagram sources into committed `.svg` files. |
| **pytest** | Test framework used for engine and adapter tests without AWS services. |
| **Quality scenario** | Testable statement of a quality requirement with source, stimulus, artifact, environment, response and measure (Chapter 10). |
| **Scale to zero** | Serverless behavior with no running or billed compute when no requests arrive. |
