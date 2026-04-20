# Snippet: Service CRUD

**Tags**: service, crud, business-logic, scaffold
**Used in**: Creating new business logic services

## Code
```go
// Location: saas/pkg/{{package}}/

package {{package}}

import (
	"context"
	"fmt"

	"dbent/gen/ent"
)

type Service struct {
	client *ent.Client
}

func NewService(client *ent.Client) *Service {
	return &Service{client: client}
}

func (s *Service) Get(ctx context.Context, id string) (*ent.{{Name}}, error) {
	result, err := s.client.{{Name}}.Get(ctx, id)
	if err != nil {
		return nil, fmt.Errorf("get {{name}} %s: %w", id, err)
	}
	return result, nil
}

func (s *Service) List(ctx context.Context) ([]*ent.{{Name}}, error) {
	results, err := s.client.{{Name}}.Query().All(ctx)
	if err != nil {
		return nil, fmt.Errorf("list {{name}}s: %w", err)
	}
	return results, nil
}

func (s *Service) Create(ctx context.Context, input Create{{Name}}Input) (*ent.{{Name}}, error) {
	result, err := s.client.{{Name}}.
		Create().
		// SetField(input.Field).
		Save(ctx)
	if err != nil {
		return nil, fmt.Errorf("create {{name}}: %w", err)
	}
	return result, nil
}

func (s *Service) Update(ctx context.Context, id string, input Update{{Name}}Input) (*ent.{{Name}}, error) {
	result, err := s.client.{{Name}}.
		UpdateOneID(id).
		// SetField(input.Field).
		Save(ctx)
	if err != nil {
		return nil, fmt.Errorf("update {{name}} %s: %w", id, err)
	}
	return result, nil
}

func (s *Service) Delete(ctx context.Context, id string) error {
	if err := s.client.{{Name}}.DeleteOneID(id).Exec(ctx); err != nil {
		return fmt.Errorf("delete {{name}} %s: %w", id, err)
	}
	return nil
}

type Create{{Name}}Input struct{}
type Update{{Name}}Input struct{}
```

## When to Use
Creating a new business logic service in `saas/pkg/`.

## Notes
- Replace `{{Name}}` (PascalCase), `{{name}}` (lowercase), `{{package}}` (package name)
- Constructor takes `*ent.Client` — get from `app.GetPlugins().EntDB.Client`
- All methods: context first, error last, wrap errors with context
