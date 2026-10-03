# AGENTGUARD MASTERPLAN v1.0

**Status:** LOCKED FOR IMPLEMENTATION  
**Repository:** https://github.com/macbro77/agentguard  
**Project Type:** Serious portfolio project  
**Primary implementation worker:** Claude via Bluehost  
**Architecture/source-of-truth document:** `MASTERPLAN.md`  
**Implementation source of truth:** GitHub repository  
**Current-state source of truth:** `PROJECT_STATUS.md`

---

# 1. Executive Summary

AgentGuard is a serious portfolio project demonstrating an infrastructure layer for AI-agent reliability, security, observability, and evaluation.

AgentGuard sits at the boundary between an AI agent and the tools/actions it attempts to execute.

The core lifecycle is:

```text
AI Agent
   ↓
AgentGuard SDK / Integration
   ↓
Tool Request
   ↓
Policy + Security Decision
   ├── DENY → Security Event → Trace
   └── ALLOW → Tool Execution → Trace
                              ↓
                         Evaluation
                              ↓
                     Regression Test
```

The project is **portfolio-first**, not startup-first.

The goal is to demonstrate:

- serious backend engineering
- security engineering
- AI-agent infrastructure
- observability
- evaluation/testing
- API design
- database design
- DevOps/infrastructure
- architecture and tradeoff reasoning

This project does **not** need to compete with LangSmith, Langfuse, Arize, AWS AgentCore, Microsoft Agent 365, or similar products.

The first release is an MVP intended to be realistically implementable by Claude in approximately ten days of focused implementation effort.

**Ten days is a target, not a hard architectural constraint.**

If Claude reaches its usage limit, a new Claude session must resume from:

1. `MASTERPLAN.md`
2. `PROJECT_STATUS.md`
3. Git history
4. existing implementation

---

# 2. Project Objective

Build a working system that can:

1. connect to an AI agent,
2. intercept a consequential tool call,
3. evaluate deterministic security/policy rules,
4. allow or deny the action,
5. record the execution,
6. expose the execution through a dashboard,
7. evaluate agent behavior,
8. convert an important failure into a regression test.

The final project must be something the developer can fully explain in a technical interview.

The project succeeds through **engineering depth and coherence**, not feature count.

---

# 3. Problem

AI agents can:

- select tools incorrectly
- provide unsafe or invalid arguments
- attempt unauthorized actions
- exceed limits
- follow malicious/indirect instructions into unsafe actions
- repeatedly call tools
- fail without useful recovery
- become difficult to debug
- regress when prompts/models/tools/logic change

Traditional application monitoring and ordinary unit tests do not provide a complete lifecycle for these failures.

AgentGuard connects the lifecycle:

```text
Runtime action
    ↓
Policy/security decision
    ↓
Execution record
    ↓
Trace/investigation
    ↓
Evaluation
    ↓
Regression test
```

---

# 4. Product Boundaries

## 4.1 AgentGuard IS

- an agent runtime assurance layer
- a tool-action policy enforcement system
- an execution tracing system
- a security-event system
- an agent evaluation and regression-testing system
- a portfolio demonstration of AI-agent infrastructure

## 4.2 AgentGuard IS NOT

- a general chatbot
- an enterprise IAM replacement
- a full SIEM
- a generic observability platform
- an autonomous AI security system
- a model-training platform
- a billing platform
- a Kubernetes platform
- a full enterprise control plane
- an MCP-only product

---

# 5. Target User

Primary target:

> A developer or small engineering team building an AI agent that calls tools and needs stronger safety, debugging, observability, and evaluation workflows.

---

# 6. Primary Use Cases

## Use Case A — Block an unsafe action

The agent requests:

```text
refund.create(amount=20000)
```

Policy:

```text
Maximum refund = 5000
```

AgentGuard:

```text
DENY
reason = AMOUNT_LIMIT
```

The denial is recorded as an execution/security event.

## Use Case B — Permit a safe action

```text
refund.create(amount=2000)
```

Policy permits it.

AgentGuard:

```text
ALLOW
→ tool executes
→ result recorded
→ trace recorded
```

## Use Case C — Investigate execution

Developer opens an execution and can inspect:

- agent
- agent version
- tool
- arguments
- policy
- decision
- reason
- latency
- result/error
- execution timeline

## Use Case D — Convert failure to regression test

Developer selects an interesting/failing execution:

```text
Create Regression Test
```

AgentGuard creates a test case and dataset entry using sanitized test data.

## Use Case E — Compare agent versions

Run a deterministic test dataset against multiple agent versions and compare results.

---

# 7. MVP Scope

## 7.1 Identity and tenancy

- users
- organizations
- memberships
- projects
- tenant/project isolation
- API keys
- OWNER / ADMIN / MEMBER roles

## 7.2 Agent management

- create agent
- update agent
- list agents
- agent versions
- active version
- status/metadata

## 7.3 Tool management

- create tool
- update tool
- list tools
- tool versions
- description
- protocol/type
- input schema
- optional output schema
- risk classification
- sensitivity classification
- enabled/disabled state

## 7.4 Policy management

- create policy
- policy versioning
- validate policy
- test policy
- publish policy
- activate policy
- retire policy
- LOG_ONLY mode
- ENFORCE mode
- deterministic ALLOW/DENY
- stable reason code

## 7.5 Runtime enforcement

Runtime must:

1. authenticate request
2. resolve tenant/project/agent
3. validate request
4. identify tool
5. resolve active policy version
6. construct authorization context
7. evaluate policy
8. persist decision
9. deny or execute
10. record result/error
11. update execution
12. emit telemetry
13. return normalized response

## 7.6 Observability

Record where available:

- execution ID
- trace ID
- span ID
- agent
- agent version
- tool
- tool version
- event type
- timestamps
- policy decision
- reason code
- latency
- outcome
- error metadata
- basic token/cost metadata

## 7.7 Security events

Minimum MVP event types:

- `POLICY_DENIED`
- `UNAUTHORIZED_TOOL`
- `INVALID_ARGUMENT`
- `RATE_LIMITED`
- `EXCESSIVE_TOOL_CALLS`
- `TOOL_ERROR`

## 7.8 Evaluation

- datasets
- test cases
- deterministic evaluators
- evaluation runs
- evaluation results
- run status
- agent/version association
- pass/fail status
- basic version comparison

Initial evaluators must be deterministic and explainable.

## 7.9 Regression workflow

A selected execution becomes a reusable test case.

Historical data must be sanitized before becoming reusable test fixtures.

MVP replay means replaying sanitized recorded test inputs through the supported evaluation/test path.

It does **not** mean executing arbitrary historical customer code.

---

# 8. Explicit MVP Non-Goals

Do NOT implement these in MVP:

- Kubernetes
- Kafka
- ClickHouse
- service mesh
- multi-region infrastructure
- enterprise SSO
- complex billing
- A2A support
- many framework adapters
- TypeScript SDK
- autonomous LLM authorization
- advanced ML anomaly detection
- arbitrary uploaded customer code execution
- complex human approval workflows
- full SIEM
- production-grade cloud deployment automation
- huge distributed telemetry architecture

These belong under future evolution.

---

# 9. Architecture

## 9.1 High-level architecture

```text
                    React + TypeScript
                            │
                            ▼
                    Spring Boot API
                     /      |      \
                    /       |       \
                   ▼        ▼        ▼
             PostgreSQL   Redis    Cedar
                    \       |       /
                     \      |      /
                      ▼     ▼     ▼
                       Runtime Layer
                            │
                  ┌─────────┴─────────┐
                  ▼                   ▼
             Python SDK            HTTP
                  │                   │
                  └─────────┬─────────┘
                            ▼
                       Tool / MCP
                            │
                            ▼
                     External Action
```

Telemetry:

```text
SDK / Runtime
      ↓
OpenTelemetry
      ↓
Backend processing
      ↓
PostgreSQL for MVP
```

## 9.2 Implementation shape

The backend starts as a **modular monolith**.

Logical modules:

- auth
- tenancy
- projects
- agents
- tools
- policies
- runtime
- executions
- security
- evaluation
- audit
- observability

Do not split these into microservices merely for appearance.

A separate Python SDK exists because it is a client library consumed by agents.

A separate Python evaluation worker is allowed only if it materially simplifies the implementation.

---

# 10. Technology Stack

Initial stack:

| Area | Technology |
|---|---|
| Frontend | React + TypeScript |
| Build tooling | Vite |
| Backend | Java + Spring Boot |
| Database | PostgreSQL |
| Cache/rate limiting | Redis |
| Policy engine | Cedar |
| Agent SDK | Python |
| Telemetry | OpenTelemetry |
| Runtime | Docker / Docker Compose |
| CI/CD | GitHub Actions |
| Backend testing | JUnit + Spring testing |
| Python testing | pytest |
| API style | REST |

## Technology rules

- use supported stable releases available during implementation
- pin compatible dependency versions
- avoid unnecessary major upgrades mid-phase
- document material dependency decisions in ADRs
- do not add infrastructure simply because it is common in enterprise systems

## Runtime rule

Windows is the developer host.

Linux containers are the canonical runtime environment.

The server must not depend on Windows-native execution of Cedar or other native/runtime components.

---

# 11. Repository Structure

Target:

```text
agentguard/
├── MASTERPLAN.md
├── PROJECT_STATUS.md
├── README.md
├── CONTRIBUTING.md
├── .gitignore
├── .editorconfig
├── apps/
│   ├── api/
│   └── web/
├── sdk/
│   └── python/
├── packages/
│   └── shared-contracts/
├── tests/
│   ├── integration/
│   ├── security/
│   ├── e2e/
│   └── fixtures/
├── infrastructure/
│   └── docker/
├── docs/
│   ├── architecture/
│   ├── api/
│   ├── database/
│   ├── security/
│   ├── evaluation/
│   ├── observability/
│   ├── development/
│   └── decisions/
└── .github/
    └── workflows/
```

Do not create empty placeholder directories only to make the tree look impressive.

---

# 12. Backend Architecture

Use package-by-feature/module.

Target logical structure:

```text
api/
├── auth/
├── tenancy/
├── projects/
├── agents/
├── tools/
├── policies/
├── runtime/
├── executions/
├── security/
├── evaluations/
├── audit/
├── observability/
└── common/
```

Within modules, separate responsibilities clearly:

- controller/API
- application/service
- domain/model
- persistence
- mapping/DTOs

Avoid:

- god services
- static global state
- database access directly from controllers
- generic utility dumping grounds
- hidden cross-module dependencies

---

# 13. API Rules

- all public APIs under `/api/v1`
- JSON request/response
- consistent error model
- validation at boundaries
- server-side tenant enforcement
- stable identifiers
- UTC timestamps
- pagination where appropriate
- explicit statuses
- no secret leakage
- documented request/response contracts

Standard error shape:

```json
{
  "error": {
    "code": "POLICY_DENIED",
    "message": "Tool call denied by policy.",
    "details": {},
    "requestId": "req_123"
  }
}
```

Error `code` values are stable programmatic identifiers.

---

# 14. Minimum API Surface

Claude must define the exact DTOs, validation rules, response schemas, status codes, and error behavior as each endpoint is implemented.

## Authentication

```text
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
GET  /api/v1/auth/me
```

## Projects

```text
GET    /api/v1/projects
POST   /api/v1/projects
GET    /api/v1/projects/{projectId}
PATCH  /api/v1/projects/{projectId}
```

## Agents

```text
GET    /api/v1/projects/{projectId}/agents
POST   /api/v1/projects/{projectId}/agents
GET    /api/v1/agents/{agentId}
PATCH  /api/v1/agents/{agentId}
POST   /api/v1/agents/{agentId}/versions
```

## Tools

```text
GET    /api/v1/projects/{projectId}/tools
POST   /api/v1/projects/{projectId}/tools
GET    /api/v1/tools/{toolId}
PATCH  /api/v1/tools/{toolId}
POST   /api/v1/tools/{toolId}/versions
```

## Policies

```text
GET    /api/v1/projects/{projectId}/policies
POST   /api/v1/projects/{projectId}/policies
GET    /api/v1/policies/{policyId}
PATCH  /api/v1/policies/{policyId}
POST   /api/v1/policies/{policyId}/validate
POST   /api/v1/policies/{policyId}/test
POST   /api/v1/policies/{policyId}/publish
POST   /api/v1/policies/{policyId}/activate
POST   /api/v1/policies/{policyId}/retire
```

## Runtime

```text
POST /api/v1/runtime/tool-calls
```

## Executions

```text
GET  /api/v1/projects/{projectId}/executions
GET  /api/v1/executions/{executionId}
GET  /api/v1/executions/{executionId}/events
POST /api/v1/executions/{executionId}/create-test-case
```

## Security

```text
GET /api/v1/projects/{projectId}/security-events
GET /api/v1/security-events/{eventId}
```

## Evaluations

```text
GET  /api/v1/projects/{projectId}/datasets
POST /api/v1/projects/{projectId}/datasets
GET  /api/v1/datasets/{datasetId}/test-cases
POST /api/v1/datasets/{datasetId}/test-cases
POST /api/v1/datasets/{datasetId}/runs
GET  /api/v1/evaluation-runs/{runId}
GET  /api/v1/evaluation-runs/{runId}/results
```

---

# 15. Authentication and Authorization

## User authentication

- strong password hashing
- secure session/access-token strategy
- HTTP-only refresh cookie where browser flow needs it
- plaintext passwords are forbidden
- secrets only through environment variables/secret stores

## API keys

Runtime requests use project- or agent-scoped API keys.

Where practical:

- store only a secure hash/fingerprint
- display raw key only at creation
- never log raw keys

## RBAC

Minimum roles:

- `OWNER`
- `ADMIN`
- `MEMBER`

Authorization checks must be centralized enough to prevent inconsistent behavior.

---

# 16. Multi-Tenancy

Hierarchy:

```text
Organization
└── Project
    ├── Agents
    ├── Tools
    ├── Policies
    ├── Executions
    ├── Security Events
    └── Evaluation Data
```

Rules:

- tenant context is derived server-side
- arbitrary client-supplied tenant IDs are never trusted
- tenant ownership is enforced on every relevant query
- project boundaries are enforced
- automated cross-tenant access tests are mandatory

---

# 17. Policy Architecture

Lifecycle:

```text
DRAFT
  ↓
VALIDATE
  ↓
TEST
  ↓
PUBLISH
  ↓
ACTIVE
  ↓
RETIRE
```

Policy versions are immutable after publication.

Modes:

- `LOG_ONLY`
- `ENFORCE`

Decision must expose:

- `ALLOW` or `DENY`
- policy ID
- policy version
- stable reason code
- optional human-readable explanation
- evaluation metadata

Security decisions must not depend solely on an LLM.

---

# 18. Cedar Integration

Cedar is the deterministic authorization engine.

Define:

- principal representation
- action representation
- resource representation
- context attributes
- policy schema
- validation
- test inputs
- result translation

Illustrative mapping:

```text
principal = agent version
action    = tool.call
resource  = tool
context   = validated request metadata + tool arguments + project context
```

AgentGuard translates the incoming runtime request into Cedar input.

Cedar returns a decision.

AgentGuard maps that decision to the normalized AgentGuard decision model.

Persist stable AgentGuard reason codes.

---

# 19. Runtime Interception

Critical runtime flow:

```text
1. SDK/adapter creates request
2. authenticate
3. validate
4. resolve tenant/project/agent
5. resolve tool/version
6. create execution
7. create tool-call
8. evaluate policy
9. record decision
10. DENY:
      security event
      finalize execution
      return denial
11. ALLOW:
      execute tool adapter
      record result/error
      finalize execution
12. emit telemetry
13. return response
```

Runtime behavior must define:

- request timeout policy
- tool timeout policy
- failure behavior
- idempotency where applicable
- safe serialization
- error propagation
- correlation IDs

---

# 20. Tool Model

Minimum metadata:

- tool ID
- project ID
- stable tool name
- display name
- description
- protocol/type
- endpoint/adapter reference
- input schema
- optional output schema
- risk level
- sensitivity
- enabled state
- version
- timestamps

MVP uses a controlled adapter model.

Never allow arbitrary uploaded executable code.

---

# 21. Normalized Event Model

AgentGuard must use a framework-neutral normalized event model.

Illustrative shape:

```json
{
  "eventId": "evt_123",
  "eventType": "tool.call",
  "timestamp": "2026-10-03T12:00:00Z",
  "organizationId": "org_1",
  "projectId": "proj_1",
  "executionId": "exec_1",
  "traceId": "trace_1",
  "spanId": "span_1",
  "agent": {
    "id": "support-agent",
    "version": "14"
  },
  "tool": {
    "id": "refund.create",
    "version": "3"
  },
  "request": {
    "arguments": {
      "orderId": "123",
      "amount": 12000
    }
  },
  "decision": {
    "status": "DENY",
    "policyId": "refund-policy",
    "policyVersion": "3",
    "reasonCode": "AMOUNT_LIMIT"
  }
}
```

The persisted database representation may normalize this across multiple relational tables.

Sensitive fields must be redacted according to policy.

---

# 22. Observability

Use OpenTelemetry for:

- traces
- spans
- correlation
- latency
- errors
- token metadata
- cost metadata where supplied

Persist a normalized MVP subset into PostgreSQL.

Do not invent a proprietary telemetry protocol unnecessarily.

Future high-volume storage is a later concern.

---

# 23. Database Architecture

PostgreSQL is the MVP source of truth.

Core entities/tables:

```text
users
organizations
memberships
projects
api_keys
agents
agent_versions
tools
tool_versions
policies
policy_versions
executions
tool_calls
security_events
datasets
test_cases
evaluation_runs
evaluation_results
audit_events
```

Every table must define:

- primary key
- foreign keys
- required timestamps
- tenant/project relationships where relevant
- uniqueness constraints
- status constraints
- indexes for expected query paths

Use database migrations.

Do not manually modify schema as ordinary development practice.

---

# 24. Redis

Approved MVP uses:

- rate limiting
- short-lived caching
- short-lived coordination

Do not use Redis as primary storage.

Do not create complex queue/event architecture without an ADR.

---

# 25. Evaluation Architecture

```text
Dataset
  ↓
Test Cases
  ↓
Evaluation Run
  ↓
Evaluator
  ↓
Results
```

Test case minimum concepts:

- input
- expected behavior/assertion
- evaluator type
- optional reference output
- tags
- source execution ID

Run minimum concepts:

- dataset ID
- agent/version
- timestamps
- status
- aggregate results
- individual results

Initial evaluators are deterministic.

Potential evaluator examples:

- expected tool
- expected argument field/value
- expected allow/deny
- maximum tool-call count
- expected final state
- required response content
- latency threshold

LLM-as-judge is future work.

---

# 26. Failure-to-Regression Workflow

```text
Execution Detail
      ↓
Create Regression Test
      ↓
Sanitize execution data
      ↓
Create test case
      ↓
Attach to dataset
      ↓
Define deterministic assertions
      ↓
Run against selected agent version
      ↓
Store result
```

Store provenance:

- source execution ID
- creator
- creation timestamp
- dataset/test-case IDs

---

# 27. Frontend Architecture

React + TypeScript.

Target navigation:

```text
Dashboard
Agents
Tools
Policies
Executions
Security Events
Evaluations
Settings
```

Critical screens:

## Dashboard

- execution volume
- allow/deny counts
- recent security events
- failures
- evaluation summary

## Agents

- agent list
- versions
- active version
- recent activity

## Tools

- registry
- metadata
- versions
- risk classification

## Policies

- list
- editor
- validation
- test
- publish
- activate
- retire
- mode

## Executions

- search/filter
- agent
- version
- tool
- decision
- latency
- status

## Execution Detail

Must show:

- execution metadata
- timeline
- tool calls
- policy decision
- reason
- latency
- errors
- correlation data
- Create Regression Test action

## Security Events

- event type
- severity
- status
- context
- execution link

## Evaluations

- datasets
- test cases
- runs
- results
- agent/version comparison

Every screen must support:

- loading state
- empty state
- error state
- successful state

---

# 28. API/Frontend Contract

Use:

- centralized API client
- typed DTOs
- centralized auth
- consistent error rendering
- schema-aware validation

Do not scatter raw HTTP requests throughout components.

---

# 29. Python SDK

Responsibilities:

- configure AgentGuard URL
- configure API key
- identify agent/version
- create execution context
- instrument tool calls
- send runtime interaction
- propagate correlation IDs
- expose safe error types

Keep SDK framework-light.

Do not make LangChain/OpenAI/Google ADK a hard dependency.

Framework-specific adapters are future work.

---

# 30. Security Model

Threat categories:

- invalid credentials
- broken authorization
- tenant-isolation failure
- forged identity
- unauthorized tools
- unsafe arguments
- policy bypass
- excessive calls
- prompt-injection-driven tool requests
- sensitive data exposure
- secrets in logs
- unsafe replay

Principles:

1. deny by default where authorization requires it
2. validate external input
3. enforce tenant context server-side
4. never trust client-supplied authorization context
5. never log secrets
6. sanitize historical data
7. treat tool execution as the consequential action boundary
8. use least privilege
9. separate authentication and authorization
10. make decisions auditable

---

# 31. Logging and Errors

Use structured logging.

Correlation should include when safe:

- request ID
- execution ID
- trace ID
- project ID
- agent ID

Never log:

- passwords
- raw API keys
- secrets
- unnecessary sensitive arguments

Client-facing errors must be useful without leaking internal implementation details.

---

# 32. Health Endpoints

At minimum:

```text
GET /health/live
GET /health/ready
```

Readiness verifies dependencies required by the running component.

---

# 33. Docker Environment

Docker Compose must support:

- backend
- frontend
- PostgreSQL
- Redis

Additional services only when genuinely required.

Clean setup flow:

```text
clone
↓
copy .env.example
↓
docker compose up
↓
migrate
↓
seed
↓
open web app
↓
run demo
```

No core server infrastructure should require Windows-native setup.

---

# 34. Configuration and Secrets

Provide:

```text
.env.example
```

Never commit real credentials.

Configuration categories:

- server
- database
- Redis
- authentication
- Cedar
- telemetry
- frontend API URL

---

# 35. CI/CD

GitHub Actions should run:

1. backend build
2. backend unit tests
3. frontend build/typecheck
4. frontend tests where applicable
5. Python SDK tests
6. integration tests where practical
7. security tests where practical
8. formatting/lint checks

Deployment automation is not required for MVP.

---

# 36. Git Workflow

Recommended commit prefixes:

- `feat`
- `fix`
- `test`
- `docs`
- `refactor`
- `chore`
- `security`

Rules:

- commit coherent milestones
- avoid giant unrelated commits
- keep main buildable as practical
- update `PROJECT_STATUS.md` at phase boundaries

---

# 37. ADR Process

Location:

```text
docs/decisions/
```

Format:

```text
# ADR-XXXX: Title

Status:
Context:
Decision:
Alternatives considered:
Consequences:
```

Use ADRs for material decisions such as:

- modular monolith
- PostgreSQL telemetry
- Cedar integration
- runtime/SDK boundary
- authentication model
- evaluation architecture
- future telemetry migration

---

# 38. Implementation Order

## Phase 0 — Repository Foundation

- project manifests
- quality config
- Docker Compose skeleton
- CI skeleton
- environment templates
- documentation foundation

## Phase 1 — Backend Foundation

- Spring Boot
- configuration
- database
- migrations
- health endpoints
- errors
- API conventions

## Phase 2 — Identity and Tenancy

- users
- organizations
- memberships
- projects
- auth
- RBAC
- API keys

## Phase 3 — Agents and Tools

- agents
- versions
- tools
- tool versions
- schemas
- risk metadata

## Phase 4 — Policies

- policy model
- Cedar
- validation
- testing
- publish
- activate
- retire
- modes

## Phase 5 — Runtime

- runtime endpoint
- request validation
- resolution
- policy decision
- allow/deny
- tool adapter
- executions
- errors

## Phase 6 — Telemetry and Security

- normalized events
- OTel
- execution timeline
- security events
- audit records

## Phase 7 — Python SDK

- configuration
- API key
- context
- tool instrumentation
- correlation IDs
- errors

## Phase 8 — Evaluation

- datasets
- test cases
- deterministic evaluators
- runs
- results
- version comparison

## Phase 9 — Regression Workflow

- create test from execution
- sanitize fixture
- provenance
- evaluation

## Phase 10 — Frontend

1. auth
2. shell
3. dashboard
4. agents
5. tools
6. policies
7. executions
8. execution detail
9. security events
10. evaluations

## Phase 11 — Hardening

- integration tests
- security tests
- E2E
- seed/demo
- docs
- CI
- cleanup

Do not reorder dependencies solely to hit a calendar date.

---

# 39. Ten-Day Target

Illustrative schedule:

| Day | Work |
|---|---|
| 1 | foundation, Compose, app shells, database, Redis, CI |
| 2 | auth, tenancy, projects, API keys |
| 3 | agents, versions, tools |
| 4 | Cedar policies |
| 5 | runtime enforcement |
| 6 | telemetry, tracing, security events |
| 7 | Python SDK + integration |
| 8 | core frontend + execution detail |
| 9 | evaluation + regression |
| 10 | tests, security, demo, docs |

If incomplete, continue in another Claude session.

Never fake completion.

---

# 40. PROJECT_STATUS.md Contract

Required structure:

```markdown
# AgentGuard Project Status

## Current Phase

## Overall Completion

## Last Completed Milestone

## Working

## In Progress

## Blocked

## Known Bugs

## Architecture Changes

## Database State

## API State

## Test State

## Documentation State

## Last Claude Session

## Exact Next Steps
```

Claude must update it at every meaningful milestone.

Before a session ends with incomplete work, Claude must update:

- current state
- completed work
- incomplete work
- known issues
- exact next action
- last commit

---

# 41. Claude Session-Resumption Protocol

At the beginning of every session:

1. read `MASTERPLAN.md`
2. read `PROJECT_STATUS.md`
3. inspect `git status`
4. inspect recent commits
5. inspect relevant source
6. verify build/tests
7. summarize current implementation state
8. continue from the documented next step

Claude must not assume plan-complete means code-complete.

---

# 42. Claude Implementation Rules

Claude MUST:

- inspect existing code before changes
- follow project patterns
- keep modules cohesive
- avoid premature abstractions
- add tests for new behavior
- use migrations for schema changes
- validate external inputs
- preserve tenant isolation
- avoid secret leakage
- document material architectural choices
- keep API contracts synchronized
- remove obsolete code during refactors
- run relevant tests
- keep repository buildable
- update `PROJECT_STATUS.md`
- commit coherent changes

Claude MUST NOT:

- invent missing requirements
- silently redesign architecture
- bypass security controls
- skip tests just to claim progress
- commit secrets
- mark incomplete features as complete
- introduce unnecessary infrastructure

Claude may make small implementation choices when they do not alter the documented architecture or product behavior.

Material architectural changes require an ADR and explicit approval before implementation.

---

# 43. Definition of Done

A feature is complete only when:

- code exists
- API/UI behavior exists
- validation exists
- authorization exists where relevant
- persistence works
- tests exist
- documentation is updated
- errors are handled
- observability is appropriate
- no known critical security issue remains
- CI passes
- `PROJECT_STATUS.md` is accurate

MVP is complete only when the canonical demo works from a clean environment.

---

# 44. MVP Acceptance Criteria

A clean clone must demonstrate:

### Setup

- Docker Compose works
- dependencies become healthy
- migrations succeed
- frontend loads
- health endpoints pass

### Agent

- create agent
- create version

### Tool

- register refund tool

### Policy

- create and activate refund policy

### Runtime

```text
refund.create(amount=20000)
→ DENY
```

```text
refund.create(amount=2000)
→ ALLOW
→ tool executes
```

### Investigation

Developer can view the execution and policy decision.

### Security

Denied action appears as a security event.

### Regression

Denied execution can create a regression test.

### Evaluation

Regression test can be evaluated deterministically.

### Version comparison

Two agent versions can be compared through evaluation results or an equivalent MVP workflow.

### Tenant security

Automated tests reject cross-tenant access.

---

# 45. Performance and Quality Targets

These are engineering benchmark targets, not commercial SLAs.

Suggested MVP targets:

- policy decision path p95 < 50 ms locally, excluding external tool latency
- 1,000 sequential simulated tool calls without functional errors
- responsive execution retrieval at demo-scale data
- deterministic evaluation is reproducible
- automated CI execution

Measure before optimizing.

---

# 46. Cost Strategy

Preferred:

- local Docker development
- open-source infrastructure
- GitHub Actions within available limits
- no paid AI service required for core MVP
- no paid observability platform
- no cloud dependency for local demo

AI-assisted evaluation is optional future work.

---

# 47. Technical Tradeoffs

## PostgreSQL instead of ClickHouse

Reason:

- simpler setup
- fewer moving parts
- sufficient for MVP/demo scale
- easier debugging

## Modular monolith instead of microservices

Reason:

- faster development
- simpler local environment
- easier testing
- easier debugging
- still preserves strong architecture boundaries

## Deterministic policies before AI

Reason:

- explainable
- testable
- secure
- predictable

## Python SDK first

Reason:

- fast implementation
- good fit for agent prototypes
- framework-neutral
- demonstrates SDK design

## Docker/Linux runtime

Reason:

- reproducible
- avoids host-specific server setup
- reduces native dependency problems

---

# 48. Future Evolution

Potential future work:

- additional framework adapters
- deeper MCP support
- ClickHouse
- event streaming
- asynchronous workers
- advanced evaluation
- optional LLM-as-judge
- richer replay
- anomaly detection
- approval workflows
- cloud deployment
- Kubernetes
- scaling benchmarks
- additional identity providers
- policy analytics

Future work must not leak into MVP scope without an explicit decision.

---

# 49. Documentation Requirements

Required:

- README
- MASTERPLAN
- PROJECT_STATUS
- architecture overview
- API documentation
- database overview
- threat/security model
- evaluation documentation
- local setup
- testing guide
- ADRs
- demo guide

README must start with a plain-language explanation before deep technical detail.

---

# 50. Portfolio Deliverables

Final repository should contain:

- working application
- automated tests
- architecture diagrams
- threat model
- policy examples
- evaluation examples
- benchmark notes
- demo instructions
- clean README
- meaningful Git history
- ADRs

Optional later:

- demo video
- portfolio case study
- technical article
- interview question bank

---

# 51. Learning Requirements

The developer must eventually be able to explain:

- Spring Boot request lifecycle
- dependency injection
- REST design
- authentication vs authorization
- JWT/session concepts
- RBAC
- multi-tenancy
- PostgreSQL indexing and transactions
- Redis use cases
- Cedar evaluation
- tool interception
- OpenTelemetry
- traces/spans/context
- SDK architecture
- evaluation design
- regression testing
- Docker networking
- CI/CD
- threat modeling
- architecture tradeoffs

A dedicated learning/interview pass happens after implementation stabilizes.

---

# 52. Final Architectural Rule

When considering a feature, ask:

> Does this directly improve the core AgentGuard lifecycle or materially demonstrate an engineering concept worth learning?

If no, defer it.

Core lifecycle:

```text
OBSERVE
+
CONTROL
+
EVALUATE
+
REGRESS
```

---

# 53. Final Source-of-Truth Rules

- `MASTERPLAN.md` = intended architecture and requirements
- `PROJECT_STATUS.md` = current implementation state
- GitHub code = actual implementation
- ADRs = approved material architectural changes
- Claude = implementation worker
- Project owner = decision maker for material changes
- No Claude session may silently redefine the product

AgentGuard v1.0 is ready for implementation.
