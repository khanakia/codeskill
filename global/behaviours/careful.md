---
name: careful
description: Production & sensitive code mode — extra caution on auth, payments, data, migrations
extends: default
---

# Careful Mode

## Overrides
- Confirm approach with user before implementing
- Run tests after every change, not just at the end
- Flag any change that touches auth, payments, user data, or encryption
- Double-check generated code paths before editing — verify `// Code generated` headers
- Check Ent schema changes for cascading impact (edges, indexes, migrations)
- Verify PKL config changes don't expose secrets

## Emphasis
- Error handling: weight 2x — every error path must be handled
- Testing: weight 2x — write tests for new code, run existing tests
- Security: weight 3x — auth middleware, API keys, encryption, CORS
- Data integrity: weight 2x — DB migrations, Ent schema changes, cascading deletes

## Sensitive Paths (auto-suggest this mode)
- `**/auth*`, `**/session*`, `**/token*`, `**/jwt*`
- `**/payment*`, `**/billing*`, `**/stripe*`
- `**/crypt*`, `**/encrypt*`, `**/password*`
- `dbent/schema/*.go` (schema changes)
- `config/pkl/**` (config changes)
- `**/middleware/**`

## Verbosity
- Explain reasoning for each decision
- List files that will be affected before starting
- Confirm approach with user before implementing
