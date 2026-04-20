# Snippet: Gin HTTP Handler

**Tags**: gin, http, handler, rest, scaffold
**Used in**: Creating REST API handlers

## Code
```go
// Location: saas/pkg/{{package}}/

package {{package}}

import (
	"net/http"

	"github.com/gin-gonic/gin"
	"go.uber.org/zap"
)

type Handler struct {
	service *Service
	logger  *zap.Logger
}

func NewHandler(service *Service, logger *zap.Logger) *Handler {
	return &Handler{service: service, logger: logger}
}

func (h *Handler) RegisterRoutes(rg *gin.RouterGroup) {
	g := rg.Group("/{{name}}s")
	{
		g.GET("", h.List)
		g.GET("/:id", h.Get)
		g.POST("", h.Create)
		g.PUT("/:id", h.Update)
		g.DELETE("/:id", h.Delete)
	}
}

func (h *Handler) List(c *gin.Context) {
	ctx := c.Request.Context()
	results, err := h.service.List(ctx)
	if err != nil {
		h.logger.Error("list failed", zap.Error(err))
		c.JSON(http.StatusInternalServerError, gin.H{"error": "internal error"})
		return
	}
	c.JSON(http.StatusOK, results)
}

func (h *Handler) Get(c *gin.Context) {
	ctx := c.Request.Context()
	result, err := h.service.Get(ctx, c.Param("id"))
	if err != nil {
		c.JSON(http.StatusNotFound, gin.H{"error": "not found"})
		return
	}
	c.JSON(http.StatusOK, result)
}

func (h *Handler) Create(c *gin.Context) {
	ctx := c.Request.Context()
	var req Create{{Name}}Request
	if err := c.ShouldBindJSON(&req); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}
	result, err := h.service.Create(ctx, Create{{Name}}Input{})
	if err != nil {
		h.logger.Error("create failed", zap.Error(err))
		c.JSON(http.StatusInternalServerError, gin.H{"error": "internal error"})
		return
	}
	c.JSON(http.StatusCreated, result)
}
```

## When to Use
Creating REST API endpoints (as alternative to GraphQL).

## Notes
- Replace `{{Name}}`, `{{name}}`, `{{package}}`
- Handler is thin: validate request → call service → return response
- Always use `c.Request.Context()` for context propagation
