---
name: debug
description: Investigation mode — hypothesis-driven, minimal changes, reproduce first
extends: default
---

# Debug Mode

## Overrides
- Form hypothesis before making any changes
- Reproduce the bug first — write a test or show the failure
- Make minimal, surgical changes only — don't "fix" adjacent code
- Verify the fix actually resolves the issue
- Check `gozap.FromCtx(ctx)` logs for trace_id to follow request flow

## Investigation Steps
1. Read the error/symptoms carefully
2. Check logs: `gozap` structured logs, trace_id, span_id
3. Check recent git changes: `git log --oneline -10`
4. Form hypothesis: "I think X is causing Y because Z"
5. Verify hypothesis with minimal test/check
6. Fix only what's broken
7. Verify fix resolves the original issue

## Tools to Use
- `gozap.FromCtx(ctx)` — structured logs with OTEL trace correlation
- Ent debug mode — enable SQL query logging
- GraphQL playground — test resolver behavior
- NATS CLI: `nats req service.endpoint '{}'` — test messaging
- `task test` — run test suite

## Emphasis
- Root cause analysis over quick fixes
- Reproduce then fix — never guess
- One change at a time — verify each step

## Verbosity
- State hypothesis explicitly before each action
- Explain what each test/check rules out
- Report root cause when found
