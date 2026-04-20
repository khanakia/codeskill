# Snippet: Ent Schema

**Tags**: ent, schema, database, scaffold
**Used in**: Creating new database entities

## Code
```go
// Location: dbent/schema/{{name}}.go
// After creating: run `task dbent:entg && task dbent:migrate`

package schema

import (
	"entgo.io/ent"
	"entgo.io/ent/schema/edge"
	"entgo.io/ent/schema/field"
	"entgo.io/ent/schema/index"

	"dbent/schema/mixin"
)

type {{Name}} struct {
	ent.Schema
}

func ({{Name}}) Mixin() []ent.Mixin {
	return []ent.Mixin{
		mixin.BaseMixin{Prefix: "xxx"}, // Change prefix (3 chars)
	}
}

func ({{Name}}) Fields() []ent.Field {
	return []ent.Field{
		field.String("name").NotEmpty(),
		field.String("description").Optional().Nillable(),
		field.Bool("is_active").Default(true),
		field.Enum("status").
			Values("draft", "published", "archived").
			Default("draft"),
	}
}

func ({{Name}}) Edges() []ent.Edge {
	return []ent.Edge{
		// Has many: edge.To("items", Item.Type),
		// Belongs to (explicit FK): field + edge
		// field.String("owner_id").NotEmpty(),
		// edge.To("owner", User.Type).Field("owner_id").Unique().Required(),
	}
}

func ({{Name}}) Indexes() []ent.Index {
	return []ent.Index{
		index.Fields("status"),
	}
}
```

## When to Use
Creating any new Ent entity. Replace `{{Name}}` with PascalCase entity name, `{{name}}` with lowercase.

## Notes
- Always use `BaseMixin` for id + timestamps
- Use explicit `_id` fields for edges, never `edge.From().Ref()`
- Run `task dbent:entg && task dbent:migrate` after creating
