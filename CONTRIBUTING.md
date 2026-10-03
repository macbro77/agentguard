# Contributing

AgentGuard is primarily a portfolio project. All implementation must follow MASTERPLAN.md.

## Rules

- Read MASTERPLAN.md before implementation.
- Read PROJECT_STATUS.md before continuing existing work.
- Inspect existing code before modifying it.
- Add tests with new behavior.
- Do not commit secrets.
- Do not silently change architecture.
- Keep documentation synchronized with implementation.
- Keep PROJECT_STATUS.md accurate.

## Architectural Changes

For a material architecture change:

1. Document the context and reason.
2. Create an ADR under docs/decisions/.
3. Update MASTERPLAN.md where the intended architecture changes.
4. Update PROJECT_STATUS.md.
5. Add or update tests.

Small implementation choices that preserve the existing architecture do not require an ADR.
