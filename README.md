# codeskill — AI Project Intelligence System

Tiered rules, switchable behaviours, iterative learning, exploration caching. One skill to make AI actually remember and improve.

## Structure

```
codeskill/
├── skill/                          # Install to ~/.claude/skills/codeskill/
│   ├── SKILL.md                    # All command routing
│   └── templates/                  # 11 templates (claude-md, feedback, decision, etc.)
│
├── global/                         # Copy into projects as .ai/global/
│   ├── RULES.md                    # Universal rules (~60 lines)
│   ├── taste.md                    # Global taste with @username attribution
│   ├── behaviours/                 # 5 modes (default, careful, review, debug, scaffold)
│   ├── patterns/go.md              # Go code patterns
│   ├── snippets/INDEX.md           # Cross-project snippets
│   ├── prompts/INDEX.md            # Cross-project prompts
│   └── memory/                     # Cross-project feedback/decisions
│
├── research/                       # Planning docs (not installed)
└── .gitignore
```

## How It Fits Together

- `skill/` → symlinked to `~/.claude/skills/codeskill/` → gives you `/codeskill` commands
- `global/` → manually copied into any project as `.ai/global/` → universal rules/taste/behaviours
- `/codeskill init` → creates project-specific `.ai/` files (RULES.md, HOTFIXES.md, etc.) alongside global/
- `/codeskill sync` → generates CLAUDE.md from global + project Tier 0 files

## Install

```bash
# Install the skill
ln -s /path/to/codeskill/skill ~/.claude/skills/codeskill

# Copy global rules into a project
cp -r /path/to/codeskill/global /your/project/.ai/global

# Initialize project-specific .ai/ structure
cd /your/project
/codeskill init

# Generate CLAUDE.md
/codeskill sync
```

## What `.ai/` Looks Like in a Project

```
project/
├── .ai/
│   ├── global/              # Copied from codeskill/global/ (universal rules)
│   ├── RULES.md             # Project-specific rules
│   ├── HOTFIXES.md          # Active AI mistake patterns
│   ├── taste.md             # Project taste overrides
│   ├── ARCHITECTURE.md      # System overview
│   ├── STACK.md             # Tech stack reference
│   ├── PATTERNS.md          # Code patterns
│   ├── behaviours/          # Can override global behaviours
│   ├── skills/backend/      # Domain-specific rules, patterns, snippets
│   ├── workflows/           # Step-by-step task guides
│   ├── guides/              # Deep reference docs
│   ├── snippets/            # Reusable code blocks
│   ├── prompts/             # Reusable prompt templates
│   ├── memory/              # Feedback (corrections) + decisions
│   ├── logs/                # Sessions, activity, incidents
│   ├── plans/               # Feature plans
│   ├── tasks/               # Backlog
│   ├── snapshots/           # Cached exploration results
│   ├── templates/           # Document templates
│   └── .gitignore           # Ignores logs/, decisions/, tasks/
└── CLAUDE.md                # Generated — run /codeskill sync
```

## Tier System

| Tier | What | Loaded | Token Cost |
|------|------|--------|------------|
| 0 | RULES + HOTFIXES + taste + behaviour | Always (in CLAUDE.md) | ~1080 tokens |
| 1 | ARCHITECTURE, PATTERNS, STACK, domain skills | When relevant | ~400 each |
| 2 | Workflows | On demand by task type | ~400 each |
| 3 | Guides | Deep dive only | ~2000 each |

## Commands

| Command | What It Does |
|---------|-------------|
| `init` | Scaffold `.ai/` in current project |
| `sync` | Regenerate CLAUDE.md from Tier 0 files |
| `status` | Show current state |
| `behaviour <name>` | Switch mode (default, careful, review, debug, scaffold) |
| `session start` | Load context, check staleness, report status |
| `session end` | Write session log + activity log |
| `continue` | Load CONTINUATION.md resume point |
| `save-state` | Write emergency CONTINUATION.md |
| `feedback <correction>` | Save user correction to memory |
| `decide <title>` | Log a decision to memory |
| `incident <description>` | Create incident report |
| `hotfix add <desc>` | Add to active hotfixes |
| `hotfix review` | Review stale hotfixes for graduation |
| `snapshot list` | Show all snapshots + staleness |
| `snapshot refresh <topic>` | Re-explore and update |
| `snippet <name>` | Find or create code snippet |
| `prompt <name>` | Find or create prompt template |
| `plan <title>` | Create feature plan |
| `task <description>` | Add to backlog |
| `memory search <query>` | Search feedback and decisions |
| `health` | Score system health 1-10 |
| `compress` | Auto-compress logs, memory, hotfixes |
| `recover` | Rebuild from git + code |
| `export cursor` | Generate .cursor/rules/ |
| `export copilot` | Generate copilot-instructions.md |

## Learning Loop

```
AI makes mistake → user corrects → auto-saved to memory/feedback/
                                        ↓
                              same mistake 2+ times?
                                   ↓ yes
                          promoted to HOTFIXES.md (loaded every session)
                                        ↓
                              30 days without recurrence?
                                   ↓ yes
                          graduated to RULES.md or archived
```

## Token Savings

| Feature | Savings |
|---------|---------|
| Tiered loading | ~8000 tokens/session (79% of .ai/ content never loaded) |
| Snapshots | ~10000 tokens/session (don't re-explore unchanged code) |
| Compact encoding | ~400 tokens/session (tables > prose) |
| Session windowing | ~5000 tokens (INDEX + last session, not all sessions) |
