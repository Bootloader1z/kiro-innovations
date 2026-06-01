# Team Tech Stack & Coding Standards

> **Last updated:** 2026-06-01
> **Maintainer:** Engineering Team

---

## Table of Contents

- [Tech Stack](#tech-stack)
- [Frontend (webkit-spa-prime)](#frontend-webkit-spa-prime)
- [Backend (webkit-api)](#backend-webkit-api)
- [Architecture Patterns](#architecture-patterns)
- [Frontend Structure](#frontend-structure)
- [Backend Structure](#backend-structure)
- [Coding Conventions](#coding-conventions)
- [Frontend Conventions](#frontend-conventions)
- [Backend Conventions](#backend-conventions)
- [API Design](#api-design)
- [Infrastructure & Deployment](#infrastructure--deployment)
- [Formatting & Linting Rules](#formatting--linting-rules)
- [Quick Reference](#quick-reference)

---

## Tech Stack

### Frontend (webkit-spa-prime)

| Category | Technology | Details |
|----------|-----------|---------|
| Framework | Vue 3 | Composition API + `<script setup>` |
| Language | TypeScript | Strict mode enabled |
| Build Tool | Vite 5 | Fast HMR, optimized builds |
| CSS | Tailwind CSS 3.4 | JIT mode, dark mode via `class` strategy |
| UI Library | PrimeVue 3.46 | Custom Tailwind presets |
| State Management | Pinia | Setup stores with `defineStore` |
| Routing | Vue Router 4 | History mode, guards for auth/roles/MFA |
| HTTP Client | VueUse `useFetch` | Wrapped via `useApiCall` composable |
| Validation | Vuelidate | Template & model-based validation |
| Icons | Font Awesome | Via `@fortawesome/vue-fontawesome` |
| Charts | ApexCharts | Via `vue3-apexcharts` |
| Linting | ESLint + Prettier | Single quotes, no semicolons, 130 width, 2-space indent |
| Git Hooks | Husky | Pre-commit hook |

### Backend (webkit-api)

| Category | Technology | Details |
|----------|-----------|---------|
| Framework | Laravel 10 | MVC + Service layer |
| Language | PHP 8.2+ | Native enums, typed properties |
| Auth | Laravel Sanctum + JWT | Via `stechstudio/laravel-jwt` |
| Permissions | Spatie Laravel Permission 6 | Role & permission-based access |
| MFA | Google2FA + OTPHP | TOTP-based multi-factor auth |
| Queue | Redis (predis) | Queues: `otp`, `emails`, `dev_alerts`, `default` |
| Storage | AWS S3 | Via `league/flysystem-aws-s3-v3` |
| Code Style | Laravel Pint | PSR-12 based |
| Static Analysis | GrumPHP | Pint + Enlightn Security Checker + PHPUnit |
| Testing | PHPUnit 10 | Unit & feature tests |
| Containerization | Docker Compose | app, queue worker, scheduler, MySQL, Redis |
| CI/CD | GitLab CI | `.gitlab-ci.yml` pipeline |
| API Versioning | URI prefix | `/v1/` |

---

## Architecture Patterns

### Frontend Structure

```
src/
├── components/             # Feature-scoped component folders
│   ├── auth-page/          #   e.g., LoginForm.vue, MfaPrompt.vue
│   ├── dashboard/          #   e.g., StatsCard.vue, ActivityFeed.vue
│   └── webkit/             #   Reusable Wb-prefixed wrappers
│       ├── WbInputText.vue
│       ├── WbDropdown.vue
│       └── ...
├── composables/            # Shared logic
│   ├── network.ts          #   useApiCall, useApiStream
│   ├── theme.ts            #   useTheme (dark/light toggle)
│   └── sidebar.ts          #   useSidebar
├── plugins/                # Vue plugin registrations
│   ├── pinia.ts
│   ├── prime-vue.ts
│   └── font-awesome.ts
├── router/                 # Single index.ts with route guards
│   └── index.ts
├── stores/                 # Pinia stores — one per domain
│   ├── auth.store.ts
│   ├── user.store.ts
│   └── notification.store.ts
├── typings/                # TypeScript interfaces and types
│   ├── auth.types.ts
│   ├── user.types.ts
│   └── api.types.ts
├── utils/                  # Pure utility functions
│   ├── helpers.ts
│   ├── error-handling.ts
│   └── validations.ts
└── views/                  # Page-level components
    ├── LoginPage.vue
    ├── DashboardPage.vue
    └── UsersPage.vue
```

### Backend Structure

```
app/
├── Auth/                   # Custom guards
│   ├── ApiKeyGuard.php
│   └── MultiTokenGuard.php
├── Enums/                  # PHP enums for constants
│   ├── Role.php
│   ├── Permission.php
│   └── ApiErrorCode.php
├── Events/                 # Domain events
│   ├── UserCreated.php
│   └── UserRegistered.php
├── Facades/                # Custom facades
│   ├── ConversionHelper.php
│   └── PaginationHelper.php
├── Helpers/                # Helper classes behind facades
├── Http/
│   ├── Controllers/        #   Thin controllers extending ApiController
│   ├── Middleware/          #   Custom middleware
│   └── Requests/           #   Form request validation classes
├── Listeners/              # Event listeners
├── Models/                 # Eloquent models
├── Notifications/          # Email/Slack notifications
├── Policies/               # Authorization policies
├── QueryFilters/           # Reusable query filter classes
├── Rules/                  # Custom validation rules
├── Services/               # Business logic (domain-scoped subdirectories)
│   ├── Auth/
│   ├── User/
│   └── Notification/
└── Traits/                 # Reusable traits
    ├── Controllers/
    └── Services/
```

---

## Coding Conventions

### Frontend Conventions

#### Component Standards

| Rule | Convention |
|------|-----------|
| File naming | PascalCase (e.g., `LoginForm.vue`) |
| Folder structure | Feature-scoped (e.g., `components/auth-page/`) |
| SFC order | `<script>` → `<template>` → `<style>` |
| Path alias | `@/` resolves to `src/` |

#### Store Pattern

```typescript
// src/stores/auth.store.ts
export const useAuthStore = defineStore('auth', () => {
  const user = ref<User | null>(null)
  const isAuthenticated = computed(() => !!user.value)

  async function login(credentials: LoginPayload) { /* ... */ }
  function logout() { /* ... */ }

  return { user, isAuthenticated, login, logout }
})
```

#### Composable Pattern

```typescript
// src/composables/network.ts
export function useApiCall(uri: string, token: string) {
  // Always use this — never raw fetch/axios
}
```

#### Key Rules

- **API calls**: Always use `useApiCall(uri, token)` — never raw `fetch` or `axios`
- **Session storage**: Use `useStorage` from VueUse for auth state hydration
- **Route meta**: Every route must declare `authType`, `label`, and optionally `roles`
- **Store naming**: `{domain}.store.ts` using Pinia setup syntax
- **Composable naming**: `use{Name}` pattern (e.g., `useApiCall`, `useTheme`)
- **Type files**: `{domain}.types.ts` in `src/typings/`

### Backend Conventions

#### Controller Pattern

```php
class UserController extends ApiController
{
    public function __construct(private UserService $userService) {}

    public function index(ListUsersRequest $request): JsonResponse
    {
        $users = $this->userService->list($request->validated());
        return $this->success($users);
    }

    public function store(CreateUserRequest $request): JsonResponse
    {
        $user = $this->userService->create($request->validated());
        return $this->success($user, 201);
    }
}
```

#### Response Format

```json
// Success
{
  "success": true,
  "data": { "id": 1, "name": "John Doe" }
}

// Error
{
  "success": false,
  "error_code": "VALIDATION_FAILED",
  "error_message": "The email field is required."
}
```

#### Key Rules

| Rule | Convention |
|------|-----------|
| Controllers | Extend `ApiController`, use `$this->success()` / `$this->error()` |
| Business logic | Lives in `app/Services/`, injected via constructor DI |
| Validation | All in dedicated `*Request` classes |
| Route files | Split by domain: `routes/api/*.routes.php`, loaded from `routes/api.php` |
| Constants | Use PHP 8.1+ native enums |
| Side effects | Use Events/Listeners (emails, notifications, logging) |
| Authorization | Use Policies (`$this->authorize('action', $model)`) |
| Pagination | Use `PaginationHelper::formatPagination()` facade |

### API Design

| Aspect | Standard |
|--------|----------|
| Prefix | All endpoints under `/v1/` |
| Auth | Bearer token (Laravel Sanctum) |
| Rate limiting | `throttle:api-users` and `throttle:api-webhooks` |
| MFA flow | Returns `mfa_token` instead of auth token when MFA is required |
| Versioning | URI-based (`/v1/`, `/v2/`) |

#### Auth Flow

```
┌─────────┐       ┌─────────┐       ┌─────────────┐
│  Client  │──────▶│  Login  │──────▶│ MFA Required?│
└─────────┘       └─────────┘       └──────┬──────┘
                                           │         │
                                        No        Yes
                                           │         │
                                           ▼         ▼
                                    ┌──────────┐  ┌──────────────┐
                                    │auth_token │  │  mfa_token   │
                                    └──────────┘  └──────┬───────┘
                                                         │
                                                         ▼
                                                  ┌─────────────┐
                                                  │ Verify TOTP  │
                                                  └──────┬──────┘
                                                         │
                                                         ▼
                                                  ┌──────────┐
                                                  │auth_token │
                                                  └──────────┘
```

---

## Infrastructure & Deployment

### Docker Compose Services

| Service | Image/Base | Purpose |
|---------|-----------|---------|
| `app` | PHP-FPM | Application server |
| `queue-worker` | PHP CLI | Processes queued jobs |
| `scheduler` | PHP CLI | Runs scheduled tasks (cron) |
| `mysql` | MySQL 8 | Primary database |
| `redis` | Redis | Cache + queue broker |

### Queue Configuration

| Queue Name | Purpose |
|-----------|---------|
| `otp` | OTP generation & delivery |
| `emails` | Transactional emails |
| `dev_alerts` | Developer notifications (Slack, etc.) |
| `default` | General background jobs |

### Deployment Pipeline

```
┌──────────┐     ┌───────────┐     ┌─────────┐     ┌──────────┐
│  GitLab  │────▶│  CI Build │────▶│   ECR   │────▶│  Deploy  │
│   Push   │     │  & Test   │     │  Push   │     │  (ECS)   │
└──────────┘     └───────────┘     └─────────┘     └──────────┘
```

- **Registry**: AWS ECR (`ap-southeast-1`)
- **Config**: Environment variables via `.env` bind mount
- **CI/CD**: GitLab CI (`.gitlab-ci.yml`)

---

## Formatting & Linting Rules

### PHP (Laravel Pint — PSR-12 based)

```json
{
  "preset": "laravel",
  "rules": {
    "no_unused_imports": true,
    "ordered_imports": { "sort_algorithm": "alpha" }
  }
}
```

### TypeScript / Vue (Prettier + ESLint)

| Rule | Value |
|------|-------|
| Quotes | Single quotes |
| Semicolons | None |
| Indent | 2 spaces |
| Print width | 130 characters |
| Trailing commas | ES5 |
| Tailwind sorting | `prettier-plugin-tailwindcss` |

```json
{
  "singleQuote": true,
  "semi": false,
  "tabWidth": 2,
  "printWidth": 130,
  "trailingComma": "es5",
  "plugins": ["prettier-plugin-tailwindcss"]
}
```

---

## Quick Reference

### Do's ✅

- Use `useApiCall` for all HTTP requests (frontend)
- Extend `ApiController` for all controllers (backend)
- Put business logic in Services, not Controllers
- Use Form Requests for validation
- Use Pinia setup stores with `defineStore`
- Declare `authType` and `label` on every route
- Use PHP native enums for constants
- Use Events/Listeners for side effects

### Don'ts ❌

- Don't use raw `fetch` or `axios` — use `useApiCall`
- Don't put business logic in controllers
- Don't skip Form Request validation
- Don't use Options API — use Composition API with `<script setup>`
- Don't hardcode constants — use Enums
- Don't bypass authorization — use Policies
- Don't commit without running Pint / ESLint + Prettier

---

*This document serves as the single source of truth for our team's technical decisions and coding standards. All new code must adhere to these conventions.*
