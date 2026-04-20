---
globs: "*.graphql,*.graphqls,**/resolver/**,**/graph/**"
description: GraphQL (gqlgen) guidelines — loaded when editing schema or resolver files
---

# GraphQL (gqlgen) Guidelines

## Schema-First
- Write schema in `*/internal/graph/schemas/*.graphql`
- Run `task <module>:gql` to generate resolvers + models
- Config in `gqlgen.yml` — `layout: follow-schema` splits files by schema

## File Locations
| What | Where |
|------|-------|
| Schema definitions | `internal/graph/schemas/*.graphql` |
| Generated code | `internal/graph/generated/` (NEVER edit) |
| Generated models | `internal/graph/model/models_gen.go` (NEVER edit) |
| Resolvers | `internal/graph/resolver/` |
| Custom directives | `internal/directives/` |

## Resolver Rules
- Resolvers ONLY in `resolver/` dir — no helpers, no utilities
- gqlgen regenerates resolver files — helpers will be deleted
- Put helpers in: `internal/<pkg>/`, `lace/`, or `saas/pkg/`
- Access services via `r.Plugin` (injected at boot)

## File Uploads
- ALWAYS base64 encoded string, NEVER `graphql.Upload`
- Validate file type by magic bytes, not filename extension
```graphql
input FileUploadInput {
  fileName: String!
  content: String!  # Base64 encoded
}
```

## Resolver Pattern — Thin Wrappers
Resolvers should be thin: validate → authenticate → call service → handle errors.
No business logic in resolvers — put it in `saas/pkg/` services.

## Directives
- `@internal` — admin-only endpoints, validated by `X-Internal-Key` header
- `@auth` — requires authenticated user
- `@role(required: "admin")` — role-based access control
- Custom directives live in `internal/directives/`

## Type Mapping
- `ID` → `graphql.ID` or `graphql.Int64`
- `Date` → `lace/gqlmodel.Date`
- `JSON` → `lace/jsontype.JSON`
- `Node` → `dbent/gen/ent.Noder`
- Ent types auto-bound via `autobind` in `gqlgen.yml` (no manual converters needed)

## After Schema Changes
1. Edit `schemas/*.graphql`
2. Run `task <module>:gql`
3. Implement new resolvers in `resolver/*.resolvers.go`
4. Test with GraphQL playground
