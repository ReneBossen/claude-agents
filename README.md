# Claude Agents Plugin

Reusable Claude Code agents and skills for software development projects following Screaming Architecture, SOLID principles, and modern best practices.

## Key Feature: Dynamic Skill Routing

**Agents don't load skills automatically.** The orchestrator (main Claude conversation) analyzes each task and injects relevant skills into subagent prompts at spawn time.

This means:
- Skills are selected based on the actual task, not static configuration
- Agents stay lightweight with minimal baseline prompts
- The orchestrator has full control over what knowledge each agent receives

## Installation

### From GitHub

```bash
claude plugin install github:your-username/claude-agents-plugin
```

### From Local Path (Development)

```bash
claude plugin install --plugin-dir /path/to/claude-agents-plugin
```

## Agents

| Agent | Description | Tools |
|-------|-------------|-------|
| `backend-engineer` | .NET Web API development | Read, Edit, Write, Bash, Grep, Glob, WebSearch, WebFetch |
| `frontend-engineer` | React Native/Expo mobile development | Read, Edit, Write, Bash, Grep, Glob, WebSearch, WebFetch |
| `database-engineer` | PostgreSQL/Supabase schema and security | Read, Grep, Glob, Bash, Write, Edit |
| `architecture-engineer` | Design decisions and documentation | Read, Write, Edit, Grep, Glob, Bash, WebSearch |
| `planner` | Feature planning and design | Read, Grep, Glob, Bash, Write, Edit (plan mode) |
| `tester` | Test creation and validation | Read, Edit, Write, Bash, Grep, Glob |
| `reviewer` | Code review (read-only) | Read, Grep, Glob, Bash |

## Skills

| Skill | Description |
|-------|-------------|
| `skill-index` | Index of all skills - orchestrator reads this first |
| `screaming-architecture` | Vertical slice architecture patterns |
| `solid-principles` | SOLID principles with examples |
| `supabase-patterns` | Database patterns for Supabase/PostgreSQL |
| `agent-contract` | Non-negotiable agent behavior rules |
| `forbidden-actions` | Prohibited actions for safety |
| `self-training` | How agents propose new skills |

## How Skill Routing Works

1. User requests a task
2. Orchestrator analyzes what's needed
3. Orchestrator reads `skill-index/SKILL.md` to identify relevant skills
4. Orchestrator reads each relevant skill file
5. Orchestrator spawns subagent with skill content prepended to the prompt

### Example Prompt Structure

```markdown
## Required Knowledge

[Content of screaming-architecture/SKILL.md]
[Content of solid-principles/SKILL.md]
[Content of agent-contract/SKILL.md]

---

## Your Task

Build the user profile API endpoint...
```

## Self-Training Feature

All agents are configured to detect and propose new skills when they identify reusable patterns. This creates a self-improving system.

### How It Works

1. Agents complete their work
2. They review what they learned
3. If they identify a reusable pattern, they propose a skill
4. You review and approve skill proposals
5. New skills are added to the plugin

### Skill Proposal Format

```markdown
## Skill Proposal

**Name**: api-response-pattern
**Category**: patterns
**Applies to**: backend-engineer

### Summary
Standard response envelope used for all API endpoints.

### Content Preview
- ApiResponse<T> wrapper structure
- Error handling patterns

### Justification
- Used in every controller endpoint
- Ensures consistent client experience
```

## Customization

### Adding Project-Specific Skills

Create skills in your project's `.claude/skills/` directory. They will be available alongside plugin skills.

### Overriding Plugin Agents

Create agents with the same name in your project's `.claude/agents/` directory to override plugin agents.

## Directory Structure

```
claude-agents-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── agents/
│   ├── backend-engineer.md
│   ├── frontend-engineer.md
│   ├── database-engineer.md
│   ├── architecture-engineer.md
│   ├── planner.md
│   ├── tester.md
│   └── reviewer.md
├── skills/
│   ├── screaming-architecture/
│   │   └── SKILL.md
│   ├── solid-principles/
│   │   └── SKILL.md
│   ├── supabase-patterns/
│   │   └── SKILL.md
│   ├── agent-contract/
│   │   └── SKILL.md
│   ├── forbidden-actions/
│   │   └── SKILL.md
│   └── self-training/
│       └── SKILL.md
└── README.md
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Add or modify agents/skills
4. Submit a pull request

## License

MIT
