---
globs: "**/app/**/*.tsx,**/pages/**/*.tsx,next.config.*"
description: Next.js guidelines — loaded when editing Next.js app or pages files
---

# Next.js Guidelines

## Images
- Use `<Image>` from `next/image`, never raw `<img>`
- Always set `width`/`height` or use `fill` prop
- Use `priority` for above-the-fold images

## Head
- Use `<Head>` from `next/head` in pages router
- Use `metadata` export in app router — never manual `<head>`

## Routing
- App router: `app/` directory with `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`
- Dynamic routes: `[param]` folders
- Route groups: `(group)` folders for organization without URL impact

## Data Fetching
- App router: `async` server components fetch directly
- Client components: TanStack Query for client-side data
- Never fetch in `useEffect` — use a data fetching library

## Document
- Never import from `next/document` in app router
- Custom `_document.tsx` only in pages router
