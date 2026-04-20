---
globs: "**/nats**,**/natso/**,**/pubsub/**"
description: NATS messaging guidelines — loaded when editing NATS handler files
---

# NATS Messaging Guidelines

## Architecture
- Uses NATS micro service pattern (gRPC-like)
- Services registered with `micro.AddService()`
- Endpoints grouped under service name: `myservice.endpoint`

## Handler Pattern
```go
srv, _ := micro.AddService(nc, micro.Config{
    Name:    "servicename",
    Version: "0.0.1",
})
m := srv.AddGroup("servicename")

m.AddEndpoint("action", micro.HandlerFunc(func(req micro.Request) {
    var input ActionInput
    json.Unmarshal(req.Data(), &input)
    // process
    req.Respond(responseBytes)
}))
```

## Event Subscription
```go
nc.Subscribe("servicename.entity.event", func(m *nats.Msg) {
    var data EventPayload
    json.Unmarshal(m.Data, &data)
    // handle event
})
```

## Registration
- Register handlers in `WithAfterNatsInitiated` callback (NATS must be initialized first)
- Handler function signature: `func handleEvent(plugin *app.Plugin, msg *nats.Msg)`

## JetStream Acknowledgment
- `msg.Ack()` — success, remove from queue
- `msg.Nak()` — retry, redeliver
- `msg.Term()` — poison message, don't retry
- `msg.InProgress()` — extend timeout, still working

## Tracing
- Add spans to handlers: span name `nats.<subject>`
- Propagate trace context via message headers

## Subject Naming
- Pattern: `{service}.{entity}.{action}`
- Examples: `thewildaigo.workspace.user.created`, `thewildaigo.ping`

## Testing NATS
```bash
nats req servicename.ping '{"key":"value"}'
```
