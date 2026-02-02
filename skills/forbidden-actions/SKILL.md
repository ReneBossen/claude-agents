# Forbidden Actions

These actions are strictly prohibited. Violations trigger an immediate STOP.

## Git Operations

**NEVER:**
- Merge branches
- Work directly on main/master branch
- Rebase or rewrite history
- Delete branches
- Use `--force` or `--force-with-lease`
- Amend commits that have been pushed

**ALWAYS:**
- Check current branch before starting work
- Create a feature branch if on main/master
- Use descriptive branch names

## Code Modifications

**NEVER:**
- Change unrelated code "while here"
- Refactor unless explicitly requested
- Rename public types unless in the plan
- Modify architecture policies
- Add/remove/update packages without approval
- Introduce dependencies without documentation
- Delete existing tests

## Scope Violations

**NEVER:**
- Invent requirements
- Assume unstated requirements
- Expand scope beyond approved plan
- Make unrequested "improvements"
- Add features not in the plan
- Change API contracts without instruction

## Safety Violations

**NEVER:**
- Bypass failing tests
- Disable analyzers or warnings
- Hardcode secrets or credentials
- Ignore security vulnerabilities
- Skip validation

## Process Violations

**NEVER:**
- Proceed when uncertain - STOP and ask
- Skip the review step
- Override user decisions
- Continue after a BLOCKER is identified

## When in Doubt

If any action feels ambiguous or risky: **STOP and ask for human clarification.**
