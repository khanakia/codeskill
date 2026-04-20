# Go Patterns

## Error Handling — goerr with Options
```go
// User-facing errors with structured metadata
err := goerr.New("user not found",
    goerr.WithMessageUser("We couldn't find your account"),
    goerr.WithCode("USER_NOT_FOUND"),
    goerr.WithTraceID(traceID),
)

// Standard wrapping for internal errors
if err != nil {
    return fmt.Errorf("create user: %w", err)
}

// Consolidating multiple validation errors
var errorList []error
for _, err := range validationErrors {
    errorList = append(errorList, fmt.Errorf("%s is required", err.Field()))
}
return errors.Join(errorList...)
```

## Constructor — Options Pattern
```go
type config struct {
    Timeout time.Duration
    Logger  *zap.Logger
}

type Option func(*config)

func WithTimeout(t time.Duration) Option {
    return func(c *config) { c.Timeout = t }
}

func WithLogger(l *zap.Logger) Option {
    return func(c *config) { c.Logger = l }
}

func New(opts ...Option) *Service {
    cfg := &config{Timeout: 5 * time.Second} // defaults
    for _, opt := range opts { opt(cfg) }
    return &Service{cfg: cfg}
}
```

## Table-Driven Tests
```go
func TestSanitize(t *testing.T) {
    tests := []struct {
        name     string
        input    string
        want     string
        wantErr  bool
    }{
        {"valid", "normal-file.pdf", "normal-file", false},
        {"spaces", "file with spaces.docx", "file_with_spaces", false},
        {"empty", "", "", true},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Sanitize(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("error = %v, wantErr %v", err, tt.wantErr)
            }
            if got != tt.want {
                t.Errorf("got %v, want %v", got, tt.want)
            }
        })
    }
}
```

## Gin Middleware — Context Injection
```go
func AuthMiddleware(entClient *ent.Client) gin.HandlerFunc {
    return func(c *gin.Context) {
        token := c.GetHeader("Authorization")
        if token == "" {
            c.AbortWithStatusJSON(401, gin.H{"error": "missing token"})
            return
        }
        user, err := validateToken(c.Request.Context(), token, entClient)
        if err != nil {
            c.AbortWithStatusJSON(401, gin.H{"error": "invalid token"})
            return
        }
        ctx := context.WithValue(c.Request.Context(), userCtxKey{}, user)
        c.Request = c.Request.WithContext(ctx)
        c.Next()
    }
}

// Extracting from context downstream
func GetUser(ctx context.Context) *ent.User {
    u, _ := ctx.Value(userCtxKey{}).(*ent.User)
    return u
}
```

## GraphQL Resolver — Plugin Access
```go
// Resolver type aliases the shared types package
type Resolver apidashTypes.Resolver

// Resolvers access services via r.Plugin
func (r *queryResolver) User(ctx context.Context, id string) (*ent.User, error) {
    return r.Plugin.EntDB.Client.User.Get(ctx, id)
}

func (r *mutationResolver) CreateUser(ctx context.Context, input model.CreateUserInput) (*ent.User, error) {
    return r.Plugin.EntDB.Client.User.
        Create().
        SetEmail(input.Email).
        SetNillableFirstName(input.FirstName).
        Save(ctx)
}
```

## NATS Micro Service
```go
srv, _ := micro.AddService(nc, micro.Config{
    Name:    "myservice",
    Version: "0.0.1",
})
m := srv.AddGroup("myservice")

m.AddEndpoint("ping", micro.HandlerFunc(func(req micro.Request) {
    req.Respond([]byte(`{"status":"ok"}`))
}))

// Event subscription
nc.Subscribe("myservice.user.created", func(m *nats.Msg) {
    var data UserCreatedEvent
    json.Unmarshal(m.Data, &data)
    // handle event
})
```

## Cron Job Registration
```go
scheduler, _ := gocron.NewScheduler(
    gocron.WithLogger(gocronlogger.New(gozap.GetLogger())),
    gocron.WithMonitor(gocronmonitor.New(gozap.GetLogger())),
    gocron.WithStopTimeout(24 * time.Hour),
)

scheduler.NewJob(
    gocron.DurationJob(5 * time.Minute),
    gocron.NewTask(processFiles, plugin),
    gocron.WithName("processFiles"),
    gocron.WithSingletonMode(gocron.LimitModeReschedule),
)

scheduler.Start()
```

## Graceful Shutdown
```go
defer shutdown.New(
    shutdown.WithMsgShuttingDown("Shutting down..."),
    shutdown.WithMsgShutdownCompleted("Stopped."),
    shutdown.WithLogger(gozap.GetLogger()),
    shutdown.WithGracefulShutdownTimeout(0),
    shutdown.WithShutdownHandler(func(ctx context.Context) error {
        return scheduler.StopJobs()
    }),
).Listen()
```

## Structured Logging
```go
// Always use FromCtx — it adds trace_id + span_id from OpenTelemetry
gozap.FromCtx(ctx).Info("user created",
    zap.String("user_id", user.ID),
    zap.String("email", user.Email),
)

gozap.FromCtx(ctx).Error("failed to create user",
    zap.Error(err),
    zap.String("email", input.Email),
)
```

## Ent Schema with Mixins
```go
type User struct {
    ent.Schema
}

func (User) Mixin() []ent.Mixin {
    return []ent.Mixin{
        BaseMixin{},    // id, created_at, updated_at
    }
}

func (User) Fields() []ent.Field {
    return []ent.Field{
        field.String("email").NotEmpty().Unique(),
        field.String("name").Optional(),
        field.String("api_key").Optional().
            GoType(enttypes.Password("")).
            Annotations(entgql.Skip(entgql.SkipWhereInput)),
    }
}

func (User) Edges() []ent.Edge {
    return []ent.Edge{
        edge.To("workspaces", Workspace.Type),
    }
}
```

## Validation — Valgo + Playground
```go
// Valgo for business validation
func validateInput(input CreateInput) error {
    val := valgo.Is(
        valgo.String(input.Name, "Name").Not().Blank(),
        valgo.String(input.Email, "Email").Not().Blank(),
    )
    if !val.Valid() {
        return utils.ValgoFirstError(val)
    }
    return nil
}

// Playground validator for struct tags
type Input struct {
    Email string `json:"email" validate:"required,email"`
    Name  string `json:"name" validate:"required,min=2,max=100"`
}
validator := nvalidator.New()
err := validator.ValidateStruct(input)
```

## PKL Config Access
```go
// Always access via singleton
appcfg := config.Get()

// Use typed fields
port := appcfg.Server.Port
dbHost := appcfg.Database.Host
stripeKey := appcfg.App.Stripe.SecretKey
```
