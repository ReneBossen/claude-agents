# Self-Training: Skill Detection and Proposal

## Purpose

Agents should actively identify patterns, conventions, and knowledge that would benefit from being documented as reusable skills. This creates a self-improving system where agents learn from their work.

## When to Propose a New Skill

Propose a new skill when you identify:

1. **Repeated Patterns**: You've used the same approach 2+ times across different tasks
2. **Project Conventions**: Discovered coding standards or architectural decisions specific to this project
3. **Domain Knowledge**: Learned business rules or domain concepts that future tasks will need
4. **Integration Patterns**: Found effective ways to integrate with external services or libraries
5. **Error Handling**: Established patterns for handling specific types of errors
6. **Best Practices**: Identified optimization techniques or quality improvements

## Skill Proposal Format

When you identify a skill worth documenting, propose it using this format:

```markdown
## Skill Proposal

**Name**: {skill-name} (kebab-case)
**Category**: {conventions | patterns | domain | integration | best-practices}
**Applies to**: {which agents would benefit}

### Summary
{One paragraph describing what this skill captures}

### Content Preview
{Key points that would be in the skill}

### Justification
- Why this should be a skill
- How often it would be used
- What problems it prevents

### Proposed Location
`.claude/skills/{skill-name}/SKILL.md`
```

## Example Proposals

### Example 1: API Response Pattern
```markdown
## Skill Proposal

**Name**: api-response-pattern
**Category**: patterns
**Applies to**: backend-engineer

### Summary
Standard response envelope used for all API endpoints with consistent error handling.

### Content Preview
- ApiResponse<T> wrapper structure
- Success/error response factories
- Error code conventions
- HTTP status code mapping

### Justification
- Used in every controller endpoint
- Ensures consistent client experience
- Prevents ad-hoc error handling
```

### Example 2: Form Validation Rules
```markdown
## Skill Proposal

**Name**: form-validation-rules
**Category**: conventions
**Applies to**: frontend-engineer

### Summary
Standard validation patterns for user input forms in the mobile app.

### Content Preview
- Required field handling
- Email/phone format validation
- Password strength requirements
- Error message display patterns

### Justification
- Applies to all user-facing forms
- Ensures consistent UX
- Prevents duplicate validation logic
```

## Automatic Skill Detection Checklist

After completing any task, ask yourself:

- [ ] Did I discover a project convention that wasn't documented?
- [ ] Did I use a pattern that would help other agents?
- [ ] Did I learn domain knowledge that future tasks will need?
- [ ] Did I find an effective approach to a common problem?
- [ ] Would documenting this prevent mistakes in future work?

If YES to any: **Propose a skill at the end of your handoff.**

## Skill Quality Criteria

Good skills are:
- **Reusable**: Applies to multiple tasks, not just one
- **Stable**: Not likely to change frequently
- **Concrete**: Contains specific, actionable guidance
- **Scoped**: Focused on one concept, not a catch-all

## Integration with Handoffs

Include skill proposals in your handoff documents:

```markdown
## Handoff

### Work Completed
{Summary of implementation}

### Skill Proposals
{Any new skills identified during this work}

### Next Steps
{What the next agent needs to do}
```
