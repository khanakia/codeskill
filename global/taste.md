# Global Taste

## Code Philosophy
- Explicit over implicit. If it takes 2 more lines to be clear, take them. — @khanakia
- Errors are values, not exceptions. Handle them at every layer. — @khanakia
- Composition over inheritance. Small interfaces, concrete implementations. — @khanakia
- No magic. If someone reading the code can't trace the flow, it's wrong. — @khanakia
- Options pattern over config structs with 10 fields. — @khanakia

## Naming
- Functions: verb-first (`CreateUser`, `ValidateInput`, `EnsureChatbotHasAiUser`) — @khanakia
- Variables: full words, specific (`userCount` not `cnt`, `emailBody` not `body`) — @khanakia
- Packages: single-word lowercase, never generic (`goerr` not `errors`, `gozap` not `logger`) — @khanakia
- Constants: describe the value (`MaxRetries` not `RETRY_CONST`) — @khanakia
- Constructors: always `New()` or `NewX()` — @khanakia

## Structure Preferences
- Flat > nested. Max 3 levels of nesting, then refactor. — @khanakia
- Early returns > deep else chains. Guard clauses at top. — @khanakia
- Table-driven > switch statements (for tests and mappings). — @khanakia
- Group by feature, not by type (all user code together, not all handlers together). — @khanakia
- Monorepo with `go.work` — each service is its own module. — @khanakia
- `internal/` for unexported service logic, `pkg/` for shared code. — @khanakia

## Patterns I Prefer
- Options pattern for constructors (`WithLogger()`, `WithTimeout()`) — @khanakia
- Context propagation everywhere — even if you don't need it now, you will. — @khanakia
- Structured logging with zap fields, never string interpolation. — @khanakia
- Separate binaries for API, cron, CLI — not one monolith binary. — @khanakia
- Schema-first GraphQL (write .graphql, generate code) not code-first. — @khanakia
- Ent ORM with mixins for shared fields (BaseMixin for id/timestamps). — @khanakia

## What I Hate
- Speculative generalization ("might need this later") — @khanakia
- Comment-heavy code (the code should speak) — @khanakia
- Deep abstraction layers that add no value — @khanakia
- "Enterprise" patterns in a startup codebase — @khanakia
- AI-generated code that looks AI-generated (generic names, excessive comments) — @khanakia
- Global singletons with panic on nil — prefer dependency injection — @khanakia
- Hardcoded strings when a const or config value exists — @khanakia
- `fmt.Println` in production code — use the logger — @khanakia

## What I Love
- Code that reads like a story top-to-bottom — @khanakia
- Tests that serve as documentation — @khanakia
- Error messages that tell you what to do, not just what failed — @khanakia
- Small, focused PRs that do one thing well — @khanakia
- Clean shutdown with proper signal handling — @khanakia
- PKL config over scattered env vars — type-safe, structured — @khanakia
