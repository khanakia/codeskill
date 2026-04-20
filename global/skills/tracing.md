---
globs: "**/otel**,**/gotel**,**/trace**,**/instrument**"
description: OpenTelemetry tracing guidelines — loaded when editing telemetry code
---

# OpenTelemetry Tracing Guidelines

## Setup
- Use `lace/gotel` package for all telemetry setup
- Service names distinguish services in observability: `githubapp`, `wildaichat`, etc.
- Enable/disable via PKL config: `Otel.EnableLogs`, `Otel.EnableTraces`, `Otel.EnableMetrics`

## Cross-Service Trace Propagation
- W3C Trace Context standard via `traceparent` header
- Format: `00-{trace_id}-{parent_span_id}-{trace_flags}`
- Global propagator: `otel.SetTextMapPropagator(propagation.NewCompositeTextMapPropagator(...))`

## HTTP
- Client: use `otelhttp.NewTransport()` for automatic trace injection
- Server: `otel.GetTextMapPropagator().Extract(ctx, headerCarrier(headers))`

## Logging Integration
- `gozap.FromCtx(ctx)` automatically adds `trace_id` and `span_id` to log entries
- Enables correlating logs with traces in observability platform

## Span Naming
- HTTP handlers: `HTTP {method} {path}`
- NATS handlers: `nats.{subject}`
- Cron jobs: `cron.{jobName}`
- DB operations: auto-instrumented by Ent

## Span Attributes for Cron Jobs
```go
span.SetAttributes(
    attribute.String("job.name", jobName),
    attribute.String("job.type", "cron"),
    attribute.Int("items.processed", count),
    attribute.Int("items.success", success),
    attribute.Int("items.error", errCount),
)
```
