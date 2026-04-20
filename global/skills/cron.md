---
globs: "**/cron/**,**/gocron**"
description: Cron job guidelines — loaded when editing scheduler or job files
---

# Cron Job Guidelines

## Scheduler Setup
```go
scheduler, _ := gocron.NewScheduler(
    gocron.WithLogger(gocronlogger.New(gozap.GetLogger())),
    gocron.WithMonitor(gocronmonitor.New(gozap.GetLogger())),
    gocron.WithStopTimeout(24 * time.Hour),
)
```

## Job Registration
```go
scheduler.NewJob(
    gocron.DurationJob(5 * time.Minute),
    gocron.NewTask(myJobFunc, plugin),
    gocron.WithName("jobName"),
    gocron.WithSingletonMode(gocron.LimitModeReschedule),
)
```

## Rules
- Max runtime: 12 hours (system has 24h graceful shutdown as buffer)
- Always use `WithSingletonMode(LimitModeReschedule)` to prevent overlap
- Jobs must be idempotent — safe to retry if interrupted
- For long tasks: break into batches with progress tracking
- Separate binary: cron runs as `cmd/cron/main.go`, not inside API

## Graceful Shutdown
- Use pre-built binary, not `go run` (signals don't forward with `go run`)
- `WithStopTimeout(24*time.Hour)` — wait for running jobs
- Supervisor: `stopwaitsecs=86400`, `stopsignal=SIGTERM`
- Docker: `stop_grace_period: 24h`

## Monitoring
- Custom logger: `gocronlogger.New()` — wraps zap
- Custom monitor: `gocronmonitor.New()` — logs job timing, status, errors
- Optional: gocron-ui dashboard for visual monitoring
