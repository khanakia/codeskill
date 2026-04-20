# Global Rules

## Generated Code — NEVER EDIT
Before editing ANY file: check for `Code generated` header.
If present → find the source file, edit that, then run the regenerate command.

| Generated | Source | Regenerate |
|-----------|--------|------------|
| `*/gen/ent/*` | `dbent/schema/*.go` | `task dbent:entg` |
| `*/graph/generated/*` | `*/graph/schemas/*.graphql` | `task <module>:gql` |
| `*/graph/model/models_gen.go` | `*.graphql` schemas | `task <module>:gql` |
| `config/gen/*` | `config/pkl/**/*.pkl` | `task config` |

## Function Signatures
- Context always first param: `func Do(ctx context.Context, ...)`
- Error always last return: `func Do(...) (*Result, error)`
- Utility functions that don't need context can skip it

## Error Handling
- Use `goerr.New()` with options for user-facing errors
- Wrap errors with context: `fmt.Errorf("create user: %w", err)`
- Use `errors.As()` for type assertion, `errors.Join()` to consolidate multiple errors
- Never panic in libraries — return errors. Only `main()` or init can panic.
- Error messages tell the caller what to do, not just what failed

## Imports — 3 Groups
```go
import (
    "context"          // 1. stdlib (alphabetical)
    "fmt"

    "lace/gozap"       // 2. local monorepo packages
    "saas/pkg/app"

    "github.com/gin-gonic/gin"  // 3. third-party (alphabetical)
)
```

## Packages
- All lowercase, no underscores: `apidash`, `gozap`, `goerr`, `natso`
- Never generic names (`util`, `helpers`, `common`) — be specific

## Constructors — Options Pattern
```go
func New(opts ...Option) *Thing {
    cfg := &config{/* defaults */}
    for _, opt := range opts { opt(cfg) }
    return &Thing{cfg: cfg}
}
```
Use for any struct with 3+ configuration fields.

## Database Access
- ALWAYS use `app.New()` → `app.GetPlugins().EntDB.Client` for DB access
- NEVER call `ent.Open()` or `sql.Open()` directly — it skips config, encryption, pooling
- Applies to: CLI commands, cron jobs, seed scripts, tests — EVERYTHING

## GraphQL
- Schema in `*/internal/graph/schemas/*.graphql`
- Resolvers ONLY in `resolver/` — no helpers there (gqlgen overwrites)
- Helpers → `internal/<pkg>/`, `lace/`, or `saas/pkg/`
- File uploads: base64 string, NEVER `graphql.Upload`
- Use `@internal` directive for admin-only endpoints

## Logging
- Always use `gozap.FromCtx(ctx)` — it extracts trace_id + span_id from OpenTelemetry
- Never use `fmt.Println` or `log.Println` in production code
- Structured fields: `zap.String("key", val)`, `zap.Error(err)`
- Don't log AND return errors — one or the other, prevents double logging
- Log at boundaries (handlers, job start/end), not every function

## Security
- Constant-time comparison for API keys: `crypto/subtle.ConstantTimeCompare()`
- Never commit `.env`, `*.creds`, `*.pem`, `*.key` files
- Extract user from context in resolvers, don't trust client headers
- Check permissions at resolver level (not middleware only)

## Testing
- Table-driven tests with `t.Run()` subtests in `_test.go` beside source
- Use `t.Helper()` in test helper functions
- Use `t.Skipf()` when external deps unavailable (Redis, DB)
- Separate test DB index (e.g., Redis DB 1 for tests)

## Config
- PKL is the config system — never use raw env vars
- Access via `config.Get()` singleton
- Never commit API keys or secrets in PKL files

## Commits
- Present-tense conventional: `feat(scope): add user lookup`
- Never commit .env values, API keys, or credentials
- Never add AI attribution or co-author lines

## AI Behavior
- Don't add features beyond what was asked
- Don't refactor adjacent code that isn't broken
- Don't add speculative error handling for impossible scenarios
- Don't update the plan or integrate ideas unless explicitly asked
- Match existing code style, even if you'd do it differently
