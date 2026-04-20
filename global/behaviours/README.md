# Behaviours

Switchable AI personality profiles. Each file defines overrides, emphasis, and verbosity.

## Available
| Name | When to Use |
|------|-------------|
| `default` | Normal coding tasks |
| `careful` | Production code, auth, payments, data mutations |
| `review` | Code review — read-only, flag issues, don't fix |
| `debug` | Investigating bugs — hypothesis-driven, minimal changes |
| `scaffold` | New feature setup — follow workflows, use snippets |
| `docs-writer` | Writing/updating documentation — clear, accessible, example-driven |

## Usage
```
/codeskill behaviour careful
/codeskill behaviour list
```

## Adding a New Behaviour
1. Create `<name>.md` in this directory
2. Use frontmatter: name, description, extends
3. Define: Overrides, Emphasis, Verbosity
4. Run `/codeskill sync` to update CLAUDE.md
