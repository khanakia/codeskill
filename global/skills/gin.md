---
globs: "**/middleware/**,**/gin**,**/server**"
description: Gin HTTP framework guidelines — loaded when editing middleware or server files
---

# Gin HTTP Guidelines

## Middleware Pattern
```go
func MyMiddleware(dep *Dependency) gin.HandlerFunc {
    return func(c *gin.Context) {
        // pre-processing
        ctx := context.WithValue(c.Request.Context(), myKey{}, value)
        c.Request = c.Request.WithContext(ctx)
        c.Next()
        // post-processing (optional)
    }
}
```

## Context Passing
- Inject into context with `context.WithValue()` + typed key struct
- Extract downstream: `val, _ := ctx.Value(myKey{}).(*Type)`
- For GraphQL: use `gqlgenfn.GinContextFromContext(ctx)` to get gin.Context back

## Error Responses
```go
c.AbortWithStatusJSON(401, gin.H{"error": "unauthorized"})
c.AbortWithStatusJSON(400, gin.H{"error": "bad request"})
```

## Server Bootstrap — Options Pattern
```go
server := ginserver.New(
    ginserver.WithBeforeHandler(middlewares...),
    ginserver.WithAppName("myapp"),
    ginserver.WithPort("8080"),
    ginserver.WithIsProd(isProd),
)
go server.Start()
```

## CORS
- Use `util.CORSMiddleware()` — already configured for the project
