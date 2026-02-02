# Agent Contract

## Non-Negotiable Principles

This contract defines the guarantees that all agents must uphold.

## Architecture Enforcement

- **Pattern**: Screaming Architecture with Vertical Slices
- **Structure**: Feature-based folders + Common
- **Violations**: Must be reported immediately

## Dependency Direction

Within a feature slice:
```
Controller → Service → Repository → Database Client
```

Rules:
- All features can depend on `/Common`
- `/Common` must NOT depend on any feature
- Controllers contain only HTTP endpoint logic
- Services contain business logic
- Repositories handle data access

## Guarantees

- No breaking changes without explicit instruction
- No business logic in controllers
- Services contain all business logic and validation
- Repositories only handle data access
- No side effects in read operations
- No hidden dependencies (service locator, static access)

## Quality Gates

All code must pass:
- Compilation without warnings
- All existing tests
- Architecture rule validation
- Code review

## Traceability

- Every change traces back to an approved plan
- Every decision is documented
- Every assumption is listed

## Agent Behavior

### Always
- Follow policies without exception
- Be explicit about assumptions
- Produce deterministic, reviewable output
- Stop and ask when uncertain

### Never
- Invent requirements
- Change unrelated code
- Bypass tests or analyzers
- Deviate from approved plans

## Escalation Triggers

STOP immediately and ask for human clarification when:
- Requirements are ambiguous
- Instructions conflict
- Policy violations exist in current code
- Uncertain about impact
- New dependencies are needed
