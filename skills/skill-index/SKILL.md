# Skill Index

## Available Skills

Use this index to identify which skills are relevant for a task before spawning subagents.

| Skill | Description | Best For |
|-------|-------------|----------|
| `screaming-architecture` | Vertical slice architecture, feature-based organization, dependency rules | Backend, Architecture, Planner |
| `solid-principles` | SOLID principles with C# examples, dependency injection patterns | Backend, Architecture, Reviewer |
| `supabase-patterns` | RLS policies, schema conventions, entity separation, PostgreSQL patterns | Database, Backend |
| `agent-contract` | Non-negotiable agent behavior, quality gates, escalation triggers | All agents |
| `forbidden-actions` | Prohibited actions, git safety, scope violations | All agents |
| `self-training` | How to detect and propose new skills | All agents |

## Skill Selection Guide

### By Task Type

| Task | Recommended Skills |
|------|-------------------|
| **API endpoint development** | screaming-architecture, solid-principles, supabase-patterns |
| **Database schema/migration** | supabase-patterns |
| **Mobile UI development** | (project-specific frontend skills) |
| **Architecture decisions** | screaming-architecture, solid-principles |
| **Feature planning** | screaming-architecture, solid-principles |
| **Code review** | screaming-architecture, solid-principles, forbidden-actions |
| **Testing** | (project-specific testing skills) |

### By Agent

| Agent | Core Skills | Optional Skills |
|-------|-------------|-----------------|
| `backend-engineer` | screaming-architecture, solid-principles | supabase-patterns |
| `frontend-engineer` | self-training | (frontend-specific skills) |
| `database-engineer` | supabase-patterns | |
| `architecture-engineer` | screaming-architecture, solid-principles | |
| `planner` | screaming-architecture, solid-principles | |
| `tester` | self-training | |
| `reviewer` | screaming-architecture, solid-principles, forbidden-actions | |

## Skill Locations

All skills are located in:
- Plugin: `claude-agents-plugin/skills/{skill-name}/SKILL.md`
- Project-specific: `.claude/skills/{skill-name}/SKILL.md`

## Adding New Skills

When a subagent proposes a new skill, add it to:
1. The appropriate skills directory
2. This index with description and usage guidance
