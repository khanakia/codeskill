---
globs: "*.pkl,**/config/**"
description: PKL configuration guidelines — loaded when editing config files
---

# PKL Config Guidelines

## Structure
```
config/
├── pkl/
│   ├── schema/       # Type definitions (Root.pkl, App.pkl, Logger.pkl)
│   └── env/
│       ├── default/  # Local dev config
│       ├── prod/     # Production config
│       └── sample/   # Template for new environments
├── gen/              # Generated Go structs (NEVER edit)
└── go.mod
```

## Schema Files (pkl/schema/)
- Define types, constraints, defaults
- All config schemas extend `Root.pkl`

## Environment Files (pkl/env/)
- `amends "../../schema/Root.pkl"` at top
- Override values per environment
- Never commit secrets — use separate key files

## Access in Go
```go
appcfg := config.Get()  // Singleton, loaded once
port := appcfg.Server.Port
dbHost := appcfg.Database.Host
```

## After Schema Changes
1. Edit `pkl/schema/*.pkl`
2. Run `task config` to regenerate Go structs
3. Update environment files if new fields added

## Rules
- Never use raw `os.Getenv()` — use PKL config
- Never commit API keys in PKL files
- Always validate config on startup: `config/cmd/validate/`
