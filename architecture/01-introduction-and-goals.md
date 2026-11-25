# 1. Introduction and Goals

## 1.1 Requirements Overview

The Mars Rover Simulator is a small, deterministic simulation used as a kata.
A mission operator supplies a rectangular map with obstacles, a rover start
position `(x, y)` and heading (`N`, `E`, `S`, `W`), and an ordered command
sequence. The simulator applies the commands in order and reports the final
position and heading. The engine performs no I/O: a mission is a value passed
in by an adapter (CLI or HTTPS API) and the result is passed back to it.

### 1.1.1 Core Navigation Rules

- `F` moves the rover one cell forward along its current heading.
- `L` and `R` rotate the rover 90 degrees counter-clockwise and clockwise
  without changing its cell.
- If the next cell contains an obstacle, `F` is ignored: the rover stays in
  place and continues with the next command.
- If the next cell lies outside the rectangular map, `F` is ignored: the rover
  stops at the boundary cell and does not wrap around (ADR-002).
- Commands are processed sequentially; a blocked `F` is not an error and does
  not end the mission.

### 1.1.2 Inputs and Outputs

- Input: map dimensions, obstacle coordinates, start coordinate, heading and
  command sequence. A mission is self-contained; no state is kept between
  requests.
- Output: final coordinate and heading of the rover.

## 1.2 Quality Goals

| Priority | Quality Goal | Scenario |
| --- | --- | --- |
| 1 | Correctness and determinism | For the same mission (map, start, heading, commands) the engine always returns the same final position and heading, including obstacle and boundary cases. |
| 2 | Testability | Navigation rules are covered by pytest unit tests without starting AWS, a web server or reading files. |
| 3 | Modularity and portability | The pure Python 3.12 engine has no AWS or I/O dependencies; CLI and Lambda adapters invoke the same engine with no duplicated rules. |
| 4 | Simplicity and low operating cost | The system has no database, scales to zero between missions and can be deployed with a single AWS SAM template. |
| 5 | Changeability | A new delivery channel or a rule change can be added without rewriting the engine. |

## 1.3 Stakeholders

| Role | Expectations |
| --- | --- |
| Mission Operator | Defines missions as a file or an HTTPS request and receives a trustworthy final position and heading. |
| Developer / kata participant | Gets unambiguous navigation rules, a small testable engine and clear adapter boundaries. |
| Course instructor / reviewer | Can verify the design against arc42, the kata rules and the ADRs. |
| DevOps / deployer | Can deploy and remove the whole system reproducibly with AWS SAM at minimal cost. |
