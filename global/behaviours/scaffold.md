---
name: scaffold
description: New feature scaffolding — follow workflows, use project patterns, generate boilerplate
extends: default
---

# Scaffold Mode

## Overrides
- ALWAYS check .ai/workflows/ for matching workflow before starting
- Follow workflow checklists step by step — don't skip
- Use .ai/snippets/ for code patterns instead of generating from scratch
- Use .ai/global/patterns/go.md for standard patterns
- Run code generation commands after schema changes:
  - GraphQL: `task <module>:gql`
  - Ent: `task dbent:entg`
  - Config: `task config`

## Common Scaffolding Tasks
| Task | Workflow | Key Steps |
|------|----------|-----------|
| New GraphQL endpoint | `new-endpoint.md` | Schema → gql → resolver → test |
| New Ent entity | `new-schema.md` | Schema → entg → migrate → GQL binding |
| New service | `new-service.md` | pkg/fn/ → options pattern → tests |
| New cron job | `new-cron-job.md` | Register → singleton mode → test |
| New CLI command | `new-cli-command.md` | Cobra command → app.New() → test |
| New NATS handler | `nats-handler.md` | Micro endpoint or subscription |

## Emphasis
- Workflow compliance: weight 3x — follow the checklist
- Snippet usage: weight 2x — don't reinvent patterns
- Code generation: weight 2x — always run generators after schema changes

## Verbosity
- Report which workflow is being followed
- Report which snippets/patterns are being used
- Report code generation commands being run
