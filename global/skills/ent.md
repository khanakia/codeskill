---
globs: "**/schema/*.go,**/gen/ent/**,**/ent.go"
description: Ent ORM guidelines — loaded when editing schema or ent files
---

# Ent ORM Guidelines

## Schema Structure
- Every schema uses `BaseMixin{}` (adds id, created_at, updated_at)
- Fields: use builders (`field.String("name").NotEmpty()`)
- Edges: define relationships (`edge.To("items", Item.Type)`)
- Annotations: `entgql.OrderField()`, `entgql.Skip()`, `entgql.Type()`

## Never Edit Generated Files
- `gen/ent/*` is auto-generated from `schema/*.go`
- Edit schemas → run `task dbent:entg` → commit generated output
- If you see `// Code generated` → do not touch

## Sensitive Fields
- Use `enttypes.Password("")` GoType for encrypted fields
- Skip from where input: `entgql.Skip(entgql.SkipWhereInput)`
- Skip from mutations when read-only: `Annotations(SkipMutations)`

## Edges — Never Use edge.From().Ref()
```go
// BAD — implicit foreign key via edge reference, hard to query and migrate
edge.From("project", Project.Type).Ref("files").Unique()

// GOOD — explicit foreign key field + edge
field.String("project_id").NotEmpty(),
edge.To("project", Project.Type).Field("project_id").Unique().Required(),
```
Always create an explicit `_id` field for foreign keys. Makes queries, migrations, and debugging straightforward.

## Query Patterns
```go
// Single entity
user, err := client.User.Get(ctx, id)

// Filtered query
users, err := client.User.Query().
    Where(user.EmailEQ(email)).
    All(ctx)

// With edges
user, err := client.User.Query().
    Where(user.ID(id)).
    WithWorkspaces().
    First(ctx)

// Pagination (GraphQL)
conn, err := client.User.Query().
    Paginate(ctx, after, first, nil, nil)
```

## Create/Update
```go
// Create
user, err := client.User.Create().
    SetEmail(input.Email).
    SetNillableFirstName(input.FirstName).
    Save(ctx)

// Update
user, err := user.Update().
    SetName(newName).
    Save(ctx)
```

## Ordering & Filtering — Use Generated Constants
```go
// BAD — hardcoded string
Order(ent.Desc("created_at"))

// GOOD — generated field constant
Order(ent.Desc(user.FieldCreatedAt))
```
Never hardcode field names as strings. Always use `<entity>.Field<Name>` constants.

## Schema Best Practices
- Use `BaseMixin{}` for common fields (id, created_at, updated_at with ID prefix)
- Sensitive fields: `field.String("password").Sensitive()` — hidden from logs
- Add indexes on frequently queried fields
- Common field patterns: `.NotEmpty()`, `.Unique()`, `.Default()`, `.Optional().Nillable()`, `.Enum().Values(...)`
- Encrypted fields: `GoType(enttypes.Password(""))` + `entgql.Skip(entgql.SkipWhereInput)`

## Migrations
- Schema changes → `task dbent:entg` → `task dbent:migrate`
- Migration SQL goes to `dbent/migrations/`
- Always use `WithForeignKeys(false)` — app handles referential integrity
- Ent auto-migrate does NOT support rollback — backup before production: `pg_dump`
- **Adding required field to existing table**: add as nullable first, backfill, then make required
- **Renaming fields**: add new → migrate data → remove old (across releases)
- **Adding unique constraints**: check for duplicates first, clean data, then add constraint
- Always test migration on dev DB before committing
