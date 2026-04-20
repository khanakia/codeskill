---
globs: "*.ts,*.tsx"
description: TypeScript guidelines — loaded when editing any .ts/.tsx file
---

# TypeScript Guidelines

## Type Safety
- Never use `any` — use `unknown` and narrow with type guards
- Prefer `as const` for literal types
- Use `export type` and `import type` for type-only exports/imports
- No non-null assertions (`!`) — handle null/undefined explicitly
- No enums — use `as const` objects or union types instead
- Exhaustive switch statements — handle every case, use `never` for default

## Strict Mode
- `strict: true` in tsconfig — always
- No `@ts-ignore` without explanation comment
- No `@ts-expect-error` without the actual error code

## Imports
- `import type { X }` for types (tree-shaking, no runtime cost)
- Group: external packages → internal aliases → relative imports
- No unused imports — remove them

## Functions
- Explicit return types on exported functions
- Arrow functions for callbacks and inline functions
- Named functions for top-level declarations
- Prefer `async/await` over `.then()` chains

## Promises & Async
- Always `await` promises — no floating promises
- Use `Promise.all()` for parallel work, not sequential awaits
- Handle errors with try/catch, not `.catch()` in async functions
- Never ignore promise rejections

## Variables
- `const` by default, `let` only when reassignment needed
- Never `var`
- No unused variables — remove or prefix with `_`
- Destructure when accessing 2+ properties

## Equality & Operators
- Always `===` and `!==`, never `==` or `!=`
- No bitwise operators unless intentional (easy to confuse `&` with `&&`)
- Nullish coalescing (`??`) over logical OR (`||`) for defaults

## Production Code
- No `console.log` — use a proper logger
- No `debugger` statements
- No `eval()` — ever
- No hardcoded secrets, API keys, or tokens

## Naming
- Interfaces: `PascalCase`, no `I` prefix (`User` not `IUser`)
- Types: `PascalCase`
- Constants: `UPPER_SNAKE_CASE` for true constants, `camelCase` for computed
- Components: `PascalCase` (React)
- Hooks: `use` prefix (`useAuth`, `useQuery`)
- Files: `kebab-case.ts` for utils, `PascalCase.tsx` for components

## React / JSX
- Functional components only — no class components
- Hooks at top level only — never inside conditions or loops
- Key prop required in iterators — never use array index as key
- No component definitions inside other components
- Fragments (`<>...</>`) over unnecessary wrapper divs
- Semantic HTML over ARIA when possible (`<button>` not `<div role="button">`)

## Accessibility
- All images need `alt` text (no redundant words like "image of")
- Mouse events must have keyboard equivalents
- No `accessKey` prop
- `aria-hidden` not on focusable elements
- Use semantic HTML: `<nav>`, `<main>`, `<section>`, `<article>`

## Testing
- No focused tests (`.only`) in committed code
- No disabled tests (`.skip`) without explanation
- No exports from test files
- Test file naming: `*.test.ts` or `*.spec.ts` beside source
