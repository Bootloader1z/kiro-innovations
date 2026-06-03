---
inclusion: fileMatch
fileMatchPattern: "**/*.{vue,ts,tsx,php,json}"
---

# Coding Conventions & Formatting

## Frontend Conventions

| Rule | Convention |
|---|---|
| File naming | PascalCase (`LoginForm.vue`) |
| Folders | Feature-scoped (`components/auth-page/`) |
| SFC order | `<script>` → `<template>` → `<style>` |
| Path alias | `@/` = `src/` |
| API calls | Always `useApiCall(uri, token)` — never fetch/axios |
| Stores | `{domain}.store.ts` with Pinia setup syntax |
| Composables | `use{Name}` pattern |
| Types | `{domain}.types.ts` in `src/typings/` |
| Session | `useStorage` from VueUse |
| Route meta | Must declare `authType` + `label` |

## Backend Conventions

| Rule | Convention |
|---|---|
| Controllers | Extend `ApiController`, use `$this->success()` / `$this->error()` |
| Business logic | `app/Services/` only — constructor DI |
| Validation | All in `*Request` Form Request classes |
| Routes | `routes/api/*.routes.php` split by domain |
| Constants | PHP 8.1+ native enums |
| Side effects | Events/Listeners (never direct calls) |
| Authorization | Policies (`$this->authorize()`) |
| Pagination | `PaginationHelper::formatPagination()` |
| Audit | Always queued via AuditListener — never sync |

## Formatting

### PHP — Laravel Pint
```json
{"preset":"laravel","rules":{"no_unused_imports":true,"ordered_imports":{"sort_algorithm":"alpha"}}}
```

### TS/Vue — Prettier
```json
{"singleQuote":true,"semi":false,"tabWidth":2,"printWidth":130,"trailingComma":"es5","plugins":["prettier-plugin-tailwindcss"]}
```

## Quick Reference

### Do's ✅
- `useApiCall` for all HTTP | `ApiController` for all controllers
- Services for logic | Form Requests for validation
- Pinia setup stores | `authType` + `label` on routes
- Enums for constants | Events for side effects
- Policies for auth | Queued audit logs

### Don'ts ❌
- No raw fetch/axios | No logic in controllers
- No Options API | No skipping Form Requests
- No hardcoded strings | No bypassing Policies
- No raw SQL | No sync audit writes
- No committing without Pint + ESLint
