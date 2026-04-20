---
globs: "*_test.go"
description: Testing guidelines — loaded when editing test files
---

# Testing Guidelines

## Table-Driven Tests (Always)
```go
tests := []struct {
    name    string
    input   string
    want    string
    wantErr bool
}{
    {"valid input", "foo", "bar", false},
    {"empty input", "", "", true},
}
for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        got, err := Do(tt.input)
        if (err != nil) != tt.wantErr {
            t.Errorf("error = %v, wantErr %v", err, tt.wantErr)
        }
        if got != tt.want {
            t.Errorf("got %v, want %v", got, tt.want)
        }
    })
}
```

## Test Helpers
```go
func setupTestDB(t *testing.T) *ent.Client {
    t.Helper()
    // setup
    t.Cleanup(func() { /* teardown */ })
    return client
}
```
- Always use `t.Helper()` in helper functions
- Use `t.Cleanup()` for teardown
- Use `t.Skipf()` when external deps unavailable

## What to Test
- Business logic in `pkg/fn/`
- GraphQL resolvers (input validation, edge cases)
- Cron job logic (idempotency, error handling)
- Utility functions in `lace/`

## What NOT to Test
- Generated code (`gen/ent/`, `generated/`, `models_gen.go`)
- Simple getters/setters
- Third-party library behavior

## External Dependencies
- Redis tests: use DB index 1, prefix keys with `test_`
- DB tests: use separate test database or transactions
- NATS tests: mock or use embedded server
