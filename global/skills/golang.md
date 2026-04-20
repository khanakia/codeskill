---
globs: "*.go"
description: Go language guidelines — loaded when editing any .go file
---

# Go Guidelines

## Function Signatures
- Context first: `func Do(ctx context.Context, input Input) (*Result, error)`
- Error last: always `error` as final return value
- Utilities without side effects can skip context

## Error Handling
- Wrap with context: `fmt.Errorf("create user: %w", err)`
- User-facing errors: `goerr.New("msg", goerr.WithCode("CODE"))`
- Multiple errors: `errors.Join(errList...)`
- Type check: `errors.As(err, &target)`
- Never panic in libraries
- Don't log AND return errors — do one or the other, not both (prevents double logging)
- Error prefixes follow package name: `package: operation failed: %w`

## Imports — 3 Groups (stdlib → local → third-party)
Enforced by goimports. Blank line between each group.

## Naming
- Packages: lowercase, single word, SINGULAR not plural (`httputil` not `httputils`)
- Exported: `PascalCase`
- Unexported: `camelCase`
- Constructors: `New()` or `NewX()`
- Options: `WithField()` pattern
- Struct naming by relationship: `GroupUser` (group has users), `UserImage` (user has images)

## Constructors
Options pattern for 3+ config fields:
```go
func New(opts ...Option) *Service { ... }
```
Simple struct for 1-2 fields:
```go
func New(db *ent.Client) *Service { return &Service{db: db} }
```

## Struct Tags
- JSON: `json:"fieldName"` (camelCase)
- Validate: `validate:"required,min=2"`
- Ent annotations: `Annotations(entgql.OrderField("NAME"))`

## Context
- Always propagate context through service calls
- Use `context.WithValue()` for request-scoped data (auth, trace)
- Use `context.WithTimeout()` for external calls
- Extract with typed key structs, not string keys

## Ent Field References — Never Hardcode Strings
```go
// BAD — hardcoded string, breaks silently if field renamed
Order(ent.Desc("created_at"))

// GOOD — generated constant, compile-time safe
Order(ent.Desc(user.FieldCreatedAt))
```
Always use generated field constants (`<entity>.Field<Name>`) for ordering, filtering, and selecting.

## Filelock — Lowercase Names
```go
// GOOD — lowercase, no spaces, no special chars
filelock.New("mailerlitesubscribechargegroup")

// BAD
filelock.New("MailerLite_Subscribe")
```

## Database — Disable Foreign Key Constraints
Always disable FK constraint creation in Ent migrations. The app handles referential integrity, not the database.

## Interfaces
- Accept interfaces, return concrete types
- Define interfaces near the consumer, not the implementation
- Avoid global state — use constructors, options pattern, context

## Security
- Constant-time comparison for API keys: `crypto/subtle.ConstantTimeCompare()`
- Password hashing: `bcrypt.GenerateFromPassword()` / `bcrypt.CompareHashAndPassword()`
- Never commit `.env`, `*.creds`, `*.pem`, `*.key` files
- File validation: magic bytes via `http.DetectContentType()`, never trust filename

## Logging
- Prefer `gozap.FromCtx(ctx)` — includes trace_id + span_id
- Fall back to `gozap.GetLogger()` when no context
- Log at boundaries (handlers, job start/end), not every function
- Don't log AND return errors (double logging)
- Structured fields only: `zap.String()`, `zap.Error()` — never string interpolation

## Concurrency
- Respect context cancellation in goroutines
- Use `sync.WaitGroup` or `errgroup.Group` for parallel work
- Protect shared state with channels or sync primitives
- Always set timeouts on external calls

## Lace Packages — Check Before Creating
Before creating a new utility, check if lace already has it:
`gozap`, `gotel`, `goerr`, `natso`, `cacheredis`, `ginserver`, `cli`, `crypt`, `filelock`, `publicid`, `httpreq`, `nvalidator`, `shutdown`, `gocron`, `db`, `filesystem`, `enttypes`, `jsontype`, `gqlgenfn`, `gqlmodel`
