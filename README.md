# AgentGuard

AgentGuard is a serious portfolio project for AI-agent security, reliability, observability, and evaluation.

## Core Lifecycle

AI Agent
    ↓
AgentGuard
    ↓
Tool Request
    ↓
Policy Decision
    ├── DENY → Security Event → Trace
    └── ALLOW → Tool Execution → Trace
                               ↓
                          Evaluation
                               ↓
                      Regression Test

## Project Goal

AgentGuard demonstrates strong engineering across:

- Java / Spring Boot
- React / TypeScript
- PostgreSQL
- Redis
- Cedar policy enforcement
- Python SDK development
- OpenTelemetry
- security engineering
- testing
- Docker
- CI/CD
- AI-agent evaluation

This is a portfolio project, not an attempt to compete with commercial agent platforms.

## Source of Truth

- MASTERPLAN.md — architectural source of truth
- PROJECT_STATUS.md — current implementation state
- GitHub — implementation source of truth
- docs/decisions/ — material architectural decisions

## Status

Implementation has not started yet.

Follow MASTERPLAN.md for the implementation order.

## Repository

https://github.com/macbro77/agentguard
