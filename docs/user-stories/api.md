# API

Domain: **API** (HTTP API / API Gateway and Lambda Adapter)

## API-STORY-001: Submit a mission over HTTPS and receive the result

**Priority:** 🔴 Core
**Architecture:** `architecture/03-system-scope-and-context.md`, section [3.2](../../architecture/03-system-scope-and-context.md#32-technical-context); `architecture/05-building-block-view.md`, sections [5.1](../../architecture/05-building-block-view.md#51-level-1-containers) and [5.3](../../architecture/05-building-block-view.md#53-interfaces-between-building-blocks); `architecture/06-runtime-view.md`, section [6.1](../../architecture/06-runtime-view.md#61-execute-a-mission-over-https); [ADR-001](../../architecture/09-architecture-decisions.md#adr-001-serverless-python-stack-on-aws); [ADR-003](../../architecture/09-architecture-decisions.md#adr-003-mission-and-result-json-contract)

AS A mission operator
I WANT to submit a mission over HTTPS
SO THAT I receive the rover's final cell and heading as a JSON response

### API-STORY-001-S1: Submit the canonical mission and receive the final position

**GIVEN**
* The HTTP API is deployed
* A mission JSON body with map `width` 5, `height` 4, obstacles `[[2, 2], [3, 1]]`, start `x` 1, `y` 1, heading `N`, and commands `FFLFR`

**WHEN**
* The operator sends `POST` with that JSON body to the API endpoint

**THEN**
* The response status is `200`
* The response body is `{"x": 0, "y": 3, "heading": "N"}`

### API-STORY-001-S2: Submit a mission with no obstacles

**GIVEN**
* The HTTP API is deployed
* A mission JSON body with map `width` 5, `height` 4, obstacles `[]`, start `x` 1, `y` 1, heading `N`, and commands `FFRFF`

**WHEN**
* The operator sends `POST` with that JSON body to the API endpoint

**THEN**
* The response status is `200`
* The response body is `{"x": 3, "y": 3, "heading": "E"}`

---

## API-INFRA-001.1: Deployment

**Architecture:** `architecture/07-deployment-view.md`, sections [7.1](../../architecture/07-deployment-view.md#71-deployment-diagram), [7.2](../../architecture/07-deployment-view.md#72-aws-environment), [7.4](../../architecture/07-deployment-view.md#74-building-block-placement) and [7.5](../../architecture/07-deployment-view.md#75-deployment-and-operations); [ADR-001](../../architecture/09-architecture-decisions.md#adr-001-serverless-python-stack-on-aws)

The HTTP API, Lambda function, execution role, and CloudWatch log group are defined in one AWS SAM template. The Lambda package includes the Lambda Adapter and the Navigation Engine.

### API-INFRA-001.1-S1: Deploy the API stack

**GIVEN**
* An AWS account and the AWS SAM CLI are configured
* The SAM template defines the API Gateway HTTP API route, Lambda function, IAM execution role, and stack output for the API endpoint

**WHEN**
* The operator runs `sam build` and `sam deploy`

**THEN**
* The stack deploys without errors
* The stack output contains the HTTPS API endpoint URL

## API-INFRA-001.2: Data store

**Architecture:** `architecture/05-building-block-view.md`, section [5.1](../../architecture/05-building-block-view.md#51-level-1-containers); `architecture/08-cross-cutting-concepts.md`, section [8.5](../../architecture/08-cross-cutting-concepts.md#85-statelessness-and-idempotence); [ADR-001](../../architecture/09-architecture-decisions.md#adr-001-serverless-python-stack-on-aws)

Not applicable: missions are self-contained and there is no storage container.

## API-INFRA-001.3: Event handling

**Architecture:** `architecture/05-building-block-view.md`, section [5.3](../../architecture/05-building-block-view.md#53-interfaces-between-building-blocks); `architecture/06-runtime-view.md`, section [6.1](../../architecture/06-runtime-view.md#61-execute-a-mission-over-https)

Not applicable: API Gateway invokes the Lambda adapter synchronously per request; no asynchronous events are published.

## API-INFRA-001.4: Monitoring and observability

**Architecture:** `architecture/07-deployment-view.md`, section [7.2](../../architecture/07-deployment-view.md#72-aws-environment); `architecture/08-cross-cutting-concepts.md`, section [8.3](../../architecture/08-cross-cutting-concepts.md#83-logging-and-monitoring)

The Lambda adapter uses the standard Python logger; AWS Lambda sends logs to the CloudWatch log group.

### API-INFRA-001.4-S1: Request acceptance is logged

**GIVEN**
* The API stack is deployed with CloudWatch Logs enabled

**WHEN**
* A valid mission request is accepted by the Lambda adapter

**THEN**
* A log entry recording mission acceptance and the request identifier appears in the function's CloudWatch log group

---

## API-BE-001.1: Parse the API request and execute the mission

**Architecture:** `architecture/05-building-block-view.md`, sections [5.2](../../architecture/05-building-block-view.md#52-level-2-navigation-engine) and [5.3](../../architecture/05-building-block-view.md#53-interfaces-between-building-blocks); `architecture/06-runtime-view.md`, section [6.1](../../architecture/06-runtime-view.md#61-execute-a-mission-over-https); `architecture/03-system-scope-and-context.md`, section [3.2](../../architecture/03-system-scope-and-context.md#32-technical-context); [ADR-003](../../architecture/09-architecture-decisions.md#adr-003-mission-and-result-json-contract)

### API-BE-001.1-S1: Lambda adapter maps the request to a mission and calls the engine

**GIVEN**
* An API Gateway proxy event whose body is `{"map":{"width":5,"height":4,"obstacles":[[2,2],[3,1]]},"start":{"x":1,"y":1,"heading":"N"},"commands":"FFLFR"}`

**WHEN**
* The Lambda adapter handles the event

**THEN**
* The adapter builds a mission value with map width 5, height 4, obstacles `(2,2)` and `(3,1)`, start `(1,1)` heading `N`, and commands `FFLFR`
* The adapter calls `execute(mission)` and receives result cell `(0, 3)` with heading `N`

## API-BE-001.2: Format the engine result as an HTTP response

**Architecture:** `architecture/05-building-block-view.md`, section [5.3](../../architecture/05-building-block-view.md#53-interfaces-between-building-blocks); `architecture/06-runtime-view.md`, section [6.1](../../architecture/06-runtime-view.md#61-execute-a-mission-over-https); `architecture/03-system-scope-and-context.md`, section [3.2](../../architecture/03-system-scope-and-context.md#32-technical-context)

### API-BE-001.2-S1: Return status 200 with the result JSON body

**GIVEN**
* The engine returns result cell `(0, 3)` with heading `N` for a valid mission

**WHEN**
* The Lambda adapter formats the response

**THEN**
* The HTTP response status code is `200`
* The response body is `{"x": 0, "y": 3, "heading": "N"}`

---

## API-FE-001.1: Display the mission result to the HTTP client

**Architecture:** `architecture/03-system-scope-and-context.md`, section [3.2](../../architecture/03-system-scope-and-context.md#32-technical-context); `architecture/05-building-block-view.md`, section [5.1](../../architecture/05-building-block-view.md#51-level-1-containers)

The HTTP client is the user-facing interface for API stories.

### API-FE-001.1-S1: HTTP client receives the final position JSON

**GIVEN**
* A mission JSON body with map `width` 5, `height` 4, obstacles `[[2, 2], [3, 1]]`, start `x` 1, `y` 1, heading `N`, and commands `FFLFR`

**WHEN**
* The operator submits the mission with an HTTP client to `POST` the API endpoint

**THEN**
* The client receives HTTP status `200`
* The client receives response body `{"x": 0, "y": 3, "heading": "N"}`

---

## Implementation Order

1. API-INFRA-001.1, API-INFRA-001.4
2. API-BE-001.1, API-BE-001.2
3. API-FE-001.1
4. API-STORY-001 end to end (S1, S2)
