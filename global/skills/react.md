---
globs: "*.tsx,*.jsx"
description: React guidelines — loaded when editing React component files
---

# React Guidelines

## Components
- Functional components only — no class components
- One component per file (small helpers OK)
- Props interface defined above component, named `<Component>Props`
- Destructure props in function signature

## Hooks
- Call at top level only — never inside conditions, loops, or callbacks
- Custom hooks: `use` prefix, return typed value
- Prefer `useMemo`/`useCallback` only when measurably needed (not by default)

## State
- Prefer server state (TanStack Query) over client state for fetched data
- Local state (`useState`) for UI-only concerns
- Lift state up only when siblings need it — not preemptively

## Patterns
- Early return for loading/error states before main render
- Separate data fetching from presentation (container/presentational or hooks)
- Collocate styles, tests, and types with components

## JSX
- Fragments (`<>`) over wrapper divs
- Semantic HTML: `<button>` not `<div onClick>`
- Key prop in lists — never array index
- Self-close tags with no children: `<Input />`
- Boolean props: `<Input disabled />` not `<Input disabled={true} />`

## Event Handlers
- Name: `handle<Event>` for functions, `on<Event>` for props
- Always type event parameter when not obvious

## Accessibility
- Every `<img>` has `alt`
- Every interactive element keyboard accessible
- `<label>` linked to form controls
- Semantic HTML over ARIA whenever possible
