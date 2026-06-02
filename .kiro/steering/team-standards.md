---
inclusion: auto
---

# Team Tech Stack & Coding Standards

> **Last updated:** 2026-06-02
> **Projects:** `webkit-spa-prime` (Frontend) · `webkit-api` (Backend)

For detailed documentation, see the split files in `.kiro/steering/standards/`:

- `#[[file:.kiro/steering/standards/tech-stack.md]]` — Technology choices and versions
- `#[[file:.kiro/steering/standards/frontend-spa.md]]` — SPA structure, UI/UX, design tokens, route guards
- `#[[file:.kiro/steering/standards/backend-api.md]]` — API structure, routes, security, audit, services
- `#[[file:.kiro/steering/standards/database.md]]` — Schema diagrams and table definitions
- `#[[file:.kiro/steering/standards/infrastructure.md]]` — Docker, queues, CI/CD, deployment
- `#[[file:.kiro/steering/standards/conventions.md]]` — Coding rules, formatting, quick reference

---

## Core Principles (Always Loaded)

### Architecture

```
webkit-spa-prime (Vue 3 SPA) ──HTTP/JSON──▶ webkit-api (Laravel API)
```

- Frontend: View → Store → `useApiCall` composable → API
- Backend: Middleware stack → Controller → Service → Event → Audit

### Critical Rules

- **Frontend**: Always use `useApiCall(uri, token)` — never raw fetch/axios
- **Frontend**: Composition API + `<script setup>` only — no Options API
- **Frontend**: Pinia setup stores with `defineStore()` — one per domain
- **Backend**: Controllers extend `ApiController`, stay thin, delegate to Services
- **Backend**: All validation in `*Request` Form Request classes
- **Backend**: Business logic in `app/Services/` only — never in controllers
- **Backend**: PHP 8.1+ native enums for all constants
- **Backend**: Events/Listeners for side effects — audit logging always queued
- **Backend**: Policies for resource-level authorization

### Response Shape

```json
{"success": true, "data": {...}}
{"success": false, "error_code": "...", "error_message": "..."}
```

### Formatting

- PHP: Laravel Pint (PSR-12), `no_unused_imports`, alpha-ordered imports
- TS/Vue: Prettier — single quotes, no semicolons, 2-space indent, 130-char width
