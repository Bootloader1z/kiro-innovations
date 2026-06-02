# Team Tech Stack & Coding Standards

> **Last updated:** 2026-06-02
> **Maintainer:** Engineering Team

---

## Table of Contents

- [Tech Stack](#tech-stack)
- [Frontend (webkit-spa-prime)](#frontend-webkit-spa-prime)
- [Backend (webkit-api)](#backend-webkit-api)
- [Architecture Patterns](#architecture-patterns)
- [Frontend Structure](#frontend-structure)
- [Backend Structure](#backend-structure)
- [API Routes](#api-routes)
- [Security Architecture](#security-architecture)
- [Audit System](#audit-system)
- [Coding Conventions](#coding-conventions)
- [Frontend Conventions](#frontend-conventions)
- [Backend Conventions](#backend-conventions)
- [Core Service Functions](#core-service-functions)
- [API Design](#api-design)
- [Database Schema](#database-schema)
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
├── components/
│   ├── auth-page/
│   │   ├── LoginForm.vue            # Email + password form
│   │   ├── MfaPrompt.vue            # 6-digit TOTP input
│   │   ├── ForgotPassword.vue       # Reset request form
│   │   └── ResetPassword.vue        # New password form
│   │
│   ├── dashboard/
│   │   ├── StatsCard.vue            # Single metric card (icon, value, label, trend)
│   │   ├── StatsGrid.vue            # 4-column stats row
│   │   ├── ActivityFeed.vue         # Real-time event list
│   │   ├── RevenueChart.vue         # ApexCharts area/line
│   │   └── UserGrowthChart.vue      # ApexCharts bar
│   │
│   ├── users/
│   │   ├── UserTable.vue            # PrimeVue DataTable + filters
│   │   ├── UserCreateDialog.vue     # Modal form for new user
│   │   ├── UserEditDialog.vue       # Modal form for editing
│   │   ├── UserRoleAssign.vue       # Role/permission picker
│   │   └── UserActionMenu.vue       # Context menu
│   │
│   ├── notifications/
│   │   ├── NotificationBell.vue     # Topbar bell with badge
│   │   ├── NotificationPanel.vue    # Slide-out panel
│   │   └── NotificationItem.vue     # Single notification row
│   │
│   ├── settings/
│   │   ├── ProfileForm.vue          # Name, email, avatar upload
│   │   ├── PasswordChange.vue       # Current + new password
│   │   ├── MfaSetup.vue            # QR code + verify step
│   │   └── ThemeToggle.vue          # Dark/light switch
│   │
│   ├── layout/
│   │   ├── AppTopbar.vue            # Search, notifications, avatar, theme
│   │   ├── AppSidebar.vue           # Collapsible nav menu
│   │   ├── AppFooter.vue            # Optional footer
│   │   └── AppLayout.vue            # Shell: sidebar + topbar + router-view
│   │
│   └── webkit/                      # Reusable Wb-prefixed wrappers
│       ├── WbInputText.vue          # PrimeVue InputText + label + error
│       ├── WbPassword.vue           # Password with toggle visibility
│       ├── WbDropdown.vue           # PrimeVue Dropdown + label
│       ├── WbDataTable.vue          # PrimeVue DataTable standardized
│       ├── WbDialog.vue             # PrimeVue Dialog wrapper
│       ├── WbButton.vue             # Button with loading state
│       ├── WbToast.vue              # Toast notification
│       └── WbConfirm.vue            # Confirmation dialog
│
├── composables/
│   ├── network.ts                   # useApiCall, useApiStream
│   ├── theme.ts                     # useTheme (dark/light/system)
│   ├── sidebar.ts                   # useSidebar (collapse state)
│   ├── permissions.ts               # usePermission (role/perm checks)
│   ├── pagination.ts                # usePagination (page, limit, total)
│   ├── toast.ts                     # useToast (success/error/info)
│   └── debounce.ts                  # useDebounce (search input)
│
├── stores/
│   ├── auth.store.ts                # user, token, mfaToken, login, logout
│   ├── user.store.ts                # users list, CRUD actions
│   ├── notification.store.ts        # unread count, list, markRead
│   └── settings.store.ts            # profile, preferences
│
├── typings/
│   ├── auth.types.ts                # LoginPayload, MfaPayload, AuthResponse
│   ├── user.types.ts                # User, CreateUserPayload, UserFilters
│   ├── notification.types.ts        # Notification, NotificationType enum
│   ├── api.types.ts                 # ApiResponse<T>, PaginatedResponse<T>
│   └── common.types.ts              # SelectOption, BreadcrumbItem
│
├── views/
│   ├── LoginPage.vue                # Auth layout, no sidebar
│   ├── DashboardPage.vue            # StatsGrid + charts + feed
│   ├── UsersPage.vue                # UserTable + dialogs
│   ├── NotificationsPage.vue        # Full notification history
│   ├── SettingsPage.vue             # Tabs: profile, security, appearance
│   └── NotFoundPage.vue             # 404
│
└── router/
    └── index.ts                     # Guards: auth, mfa, roles
```

### Frontend UI/UX Wireframe

```
┌─────────────────────────────────────────────────────────────────┐
│  TOPBAR                                                          │
│  ┌──────┐  ┌──────────────────────────────┐  ┌──┐ ┌──┐ ┌────┐ │
│  │ Menu │  │  Search...                    │  │Bell│ │Theme│ │Avatar│ │
│  └──────┘  └──────────────────────────────┘  └──┘ └──┘ └────┘ │
├────────┬────────────────────────────────────────────────────────┤
│SIDEBAR │  MAIN CONTENT AREA                                      │
│        │                                                         │
│  Dash  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐    │
│  Users │  │Total Users│ │Active   │ │Revenue  │ │Pending  │    │
│  Notif │  │  1,245   │ │  892    │ │ $45.2K  │ │   23    │    │
│  Sett  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘    │
│        │                                                         │
│        │  ┌─────────────────────────┐ ┌───────────────────────┐ │
│        │  │   Area Chart            │ │   Activity Feed       │ │
│        │  │   (ApexCharts)          │ │   - User X logged in  │ │
│        │  │                         │ │   - Role Y assigned   │ │
│        │  └─────────────────────────┘ └───────────────────────┘ │
└────────┴────────────────────────────────────────────────────────┘
```

### Frontend Design Tokens

| Token | Light | Dark |
|-------|-------|------|
| Background | `#FFFFFF` | `#1E1E2E` |
| Surface | `#F8FAFC` | `#2A2A3C` |
| Primary | `#3B82F6` | `#60A5FA` |
| Sidebar BG | `#1E293B` | `#0F172A` |
| Text | `#1E293B` | `#E2E8F0` |
| Border | `#E2E8F0` | `#334155` |
| Success | `#10B981` | `#34D399` |
| Danger | `#EF4444` | `#F87171` |

### Frontend UX Patterns

- Responsive sidebar: full on desktop, overlay on mobile, collapsible to icons
- Toast notifications: bottom-right, auto-dismiss 5s, stacked
- Skeleton loading: PrimeVue Skeleton for all data-fetching states
- Optimistic UI: toggle user status shows immediately, reverts on API failure
- Keyboard shortcuts: `/` to focus search, `Esc` to close modals
- Breadcrumbs: auto-generated from route meta `label`
- Empty states: illustrated placeholder when no data matches filter

### Route Guards Logic

```typescript
interface RouteMeta {
  authType: 'public' | 'authenticated' | 'mfa-verified'
  label: string
  roles?: string[]
  icon?: string
}
// Guard flow:
// 1. Check authType — redirect to /login if unauthenticated
// 2. Check MFA — redirect to /mfa if mfa_token but no auth_token
// 3. Check roles — redirect to /403 if insufficient permissions
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
│   ├── AuditAction.php
│   └── ApiErrorCode.php
├── Events/                 # Domain events
│   ├── UserCreated.php
│   ├── UserUpdated.php
│   ├── UserDeleted.php
│   ├── UserLoggedIn.php
│   ├── UserLoggedOut.php
│   ├── MfaVerified.php
│   ├── PasswordReset.php
│   ├── RoleAssigned.php
│   ├── StatusToggled.php
│   └── AuditExported.php
├── Facades/                # Custom facades
│   ├── ConversionHelper.php
│   └── PaginationHelper.php
├── Helpers/                # Helper classes behind facades
├── Http/
│   ├── Controllers/        #   Thin controllers extending ApiController
│   ├── Middleware/          #   Custom middleware
│   └── Requests/           #   Form request validation classes
├── Listeners/              # Event listeners
│   └── AuditListener.php   #   Auto-logs all auditable events
├── Models/                 # Eloquent models
├── Notifications/          # Email/Slack notifications
├── Policies/               # Authorization policies
├── QueryFilters/           # Reusable query filter classes
├── Rules/                  # Custom validation rules
├── Services/               # Business logic (domain-scoped subdirectories)
│   ├── Auth/
│   │   ├── AuthService.php
│   │   └── MfaService.php
│   ├── User/
│   │   └── UserService.php
│   ├── Notification/
│   │   └── NotificationService.php
│   ├── Dashboard/
│   │   └── DashboardService.php
│   └── Audit/
│       └── AuditService.php
└── Traits/                 # Reusable traits
    ├── Controllers/
    └── Services/
```

---

## API Routes

All routes prefixed with `/api/v1/`. Split by domain file under `routes/api/`.

### Auth (`routes/api/auth.routes.php`)

| Method | Endpoint | Middleware | Description |
|--------|----------|------------|-------------|
| POST | `/auth/login` | `throttle:5,1` | Email + password login |
| POST | `/auth/mfa/verify` | `throttle:3,1` | Verify TOTP code |
| POST | `/auth/refresh` | `auth:sanctum` | Refresh JWT token |
| POST | `/auth/logout` | `auth:sanctum` | Revoke token |
| POST | `/auth/forgot-password` | `throttle:3,5` | Send reset email |
| POST | `/auth/reset-password` | `throttle:3,5` | Set new password |

### Users (`routes/api/user.routes.php`)

| Method | Endpoint | Middleware | Description |
|--------|----------|------------|-------------|
| GET | `/users` | `auth:sanctum`, `role:admin` | List users (paginated + filters) |
| POST | `/users` | `auth:sanctum`, `role:admin` | Create user |
| GET | `/users/{id}` | `auth:sanctum`, `role:admin` | Get user details |
| PUT | `/users/{id}` | `auth:sanctum`, `role:admin` | Update user |
| DELETE | `/users/{id}` | `auth:sanctum`, `role:admin` | Soft-delete user |
| PATCH | `/users/{id}/status` | `auth:sanctum`, `role:admin` | Enable/disable |
| POST | `/users/{id}/roles` | `auth:sanctum`, `role:admin` | Assign roles |
| DELETE | `/users/{id}/roles` | `auth:sanctum`, `role:admin` | Remove roles |
| POST | `/users/{id}/reset-password` | `auth:sanctum`, `role:admin` | Admin force reset |

### Profile (`routes/api/profile.routes.php`)

| Method | Endpoint | Middleware | Description |
|--------|----------|------------|-------------|
| GET | `/profile` | `auth:sanctum` | Current user profile |
| PUT | `/profile` | `auth:sanctum` | Update name, email, avatar |
| PUT | `/profile/password` | `auth:sanctum` | Change own password |
| POST | `/profile/mfa/enable` | `auth:sanctum` | Generate QR + secret |
| POST | `/profile/mfa/confirm` | `auth:sanctum` | Verify first TOTP |
| DELETE | `/profile/mfa/disable` | `auth:sanctum` | Turn off MFA |

### Notifications (`routes/api/notification.routes.php`)

| Method | Endpoint | Middleware | Description |
|--------|----------|------------|-------------|
| GET | `/notifications` | `auth:sanctum` | List notifications (paginated) |
| GET | `/notifications/unread-count` | `auth:sanctum` | Badge count |
| PATCH | `/notifications/{id}/read` | `auth:sanctum` | Mark single read |
| PATCH | `/notifications/read-all` | `auth:sanctum` | Mark all read |

### Dashboard (`routes/api/dashboard.routes.php`)

| Method | Endpoint | Middleware | Description |
|--------|----------|------------|-------------|
| GET | `/dashboard/stats` | `auth:sanctum` | KPI cards data |
| GET | `/dashboard/activity` | `auth:sanctum` | Recent events feed |
| GET | `/dashboard/charts/users` | `auth:sanctum`, `role:admin` | User growth chart |
| GET | `/dashboard/charts/revenue` | `auth:sanctum`, `role:admin` | Revenue chart |

### Audits (`routes/api/audit.routes.php`)

| Method | Endpoint | Middleware | Description |
|--------|----------|------------|-------------|
| GET | `/audits` | `auth:sanctum`, `role:admin` | List audit logs (paginated) |
| GET | `/audits/{id}` | `auth:sanctum`, `role:admin` | Single audit detail |
| GET | `/audits/export` | `auth:sanctum`, `role:admin` | CSV export |

### Roles & Permissions (`routes/api/role.routes.php`)

| Method | Endpoint | Middleware | Description |
|--------|----------|------------|-------------|
| GET | `/roles` | `auth:sanctum`, `role:admin` | List all roles |
| POST | `/roles` | `auth:sanctum`, `role:admin` | Create role |
| PUT | `/roles/{id}` | `auth:sanctum`, `role:admin` | Update role |
| DELETE | `/roles/{id}` | `auth:sanctum`, `role:admin` | Delete role |
| GET | `/permissions` | `auth:sanctum`, `role:admin` | List all permissions |

---

## Security Architecture

```
Client Request
     │
     ▼
┌──────────────────┐
│  Rate Limiting    │  throttle:api-users (60/min)
└────────┬─────────┘
         ▼
┌──────────────────┐
│  CORS Middleware  │  Allowed origins from .env
└────────┬─────────┘
         ▼
┌──────────────────┐
│  Sanctum Auth     │  Bearer token validation, 60 min expiry
└────────┬─────────┘
         ▼
┌──────────────────┐
│  MFA Gate         │  Blocks if MFA enabled but not verified
└────────┬─────────┘
         ▼
┌──────────────────┐
│  Role/Permission  │  Spatie middleware: role:admin, permission:manage-users
└────────┬─────────┘
         ▼
┌──────────────────┐
│  Form Request     │  Input validation + XSS sanitization
└────────┬─────────┘
         ▼
┌──────────────────┐
│  Policy Auth      │  Resource-level access control
└────────┬─────────┘
         ▼
┌──────────────────┐
│  Controller       │  Thin — delegates to Service
└────────┬─────────┘
         ▼
┌──────────────────┐
│  Audit Logger     │  Non-blocking, queued via AuditListener
└──────────────────┘
```

### Security Layers

| Layer | Implementation | Purpose |
|-------|---------------|---------|
| Rate Limiting | `throttle` middleware | Brute-force prevention |
| CORS | `config/cors.php` | Origin whitelisting |
| Authentication | Sanctum + JWT | Token-based identity |
| MFA | Google2FA TOTP | Second factor verification |
| Authorization (role) | Spatie `role:` middleware | Role gate |
| Authorization (resource) | Laravel Policies | Per-model access control |
| Input Validation | Form Requests | Type-safe, sanitized input |
| XSS Prevention | HTMLPurifier + `strip_tags` | Output encoding |
| SQL Injection | Eloquent ORM (parameterized) | No raw queries |
| Secrets | `.env` + AWS Secrets Manager | No hardcoded credentials |
| Token Expiry | 60 min access, 7-day refresh | Automatic rotation |

### Middleware Stack

```php
// Global API middleware:
'api' => [
    ThrottleRequests::class . ':api-users',
    SubstituteBindings::class,
    ForceJsonResponse::class,
    LogRequestId::class,        // X-Request-ID header
    TrimStrings::class,
],

// Route-level:
'auth:sanctum'       // Token validation
'mfa.verified'       // MFA gate check
'role:{role}'        // Spatie role check
'permission:{perm}'  // Spatie permission check
```

---

## Audit System

### Audit Table Schema

| Column | Type | Description |
|--------|------|-------------|
| `id` | `bigint` | Primary key |
| `user_id` | `bigint nullable` | Who performed action (null = system) |
| `action` | `enum` | created, updated, deleted, login, logout, mfa_verified, password_reset, role_changed, status_changed, export |
| `auditable_type` | `string` | Polymorphic model class |
| `auditable_id` | `bigint` | Polymorphic model ID |
| `old_values` | `json nullable` | Previous state |
| `new_values` | `json nullable` | New state |
| `ip_address` | `string` | Request IP |
| `user_agent` | `string` | Browser/client info |
| `metadata` | `json nullable` | Extra context (route, request_id) |
| `created_at` | `timestamp` | When it happened |

### Auditable Events

| Event Class | Action Logged |
|-------------|---------------|
| `UserCreated` | `created` |
| `UserUpdated` | `updated` (with old/new diff) |
| `UserDeleted` | `deleted` |
| `UserLoggedIn` | `login` |
| `UserLoggedOut` | `logout` |
| `MfaVerified` | `mfa_verified` |
| `PasswordReset` | `password_reset` |
| `RoleAssigned` | `role_changed` |
| `StatusToggled` | `status_changed` |
| `AuditExported` | `export` |

### Audit Query Filters

| Filter | Type | Example |
|--------|------|---------|
| `user_id` | integer | `?user_id=5` |
| `action` | enum | `?action=login` |
| `auditable_type` | string | `?auditable_type=User` |
| `date_from` | date | `?date_from=2026-01-01` |
| `date_to` | date | `?date_to=2026-06-01` |
| `ip_address` | string | `?ip_address=192.168.1.1` |

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
{
  "success": true,
  "data": { "id": 1, "name": "John Doe" }
}
```

```json
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

---

## Core Service Functions

### AuthService (`app/Services/Auth/AuthService.php`)

```php
class AuthService
{
    public function login(array $credentials): AuthResponse;
    public function verifyMfa(string $mfaToken, string $otpCode): AuthResponse;
    public function refreshToken(User $user): TokenPair;
    public function logout(User $user): void;
    public function forgotPassword(string $email): void;
    public function resetPassword(array $data): void;
}
```

### MfaService (`app/Services/Auth/MfaService.php`)

```php
class MfaService
{
    public function generateSecret(User $user): MfaSetupData;
    public function confirmSetup(User $user, string $code): bool;
    public function verify(User $user, string $code): bool;
    public function disable(User $user): void;
    public function generateRecoveryCodes(User $user): array;
}
```

### UserService (`app/Services/User/UserService.php`)

```php
class UserService
{
    public function list(array $filters): LengthAwarePaginator;
    public function create(array $data): User;
    public function update(User $user, array $data): User;
    public function delete(User $user): void;
    public function toggleStatus(User $user): User;
    public function assignRoles(User $user, array $roles): void;
    public function revokeRoles(User $user, array $roles): void;
    public function forceResetPassword(User $user): void;
}
```

### NotificationService (`app/Services/Notification/NotificationService.php`)

```php
class NotificationService
{
    public function list(User $user, array $filters): LengthAwarePaginator;
    public function unreadCount(User $user): int;
    public function markRead(User $user, int $notificationId): void;
    public function markAllRead(User $user): void;
    public function send(User $user, NotificationType $type, array $data): void;
}
```

### DashboardService (`app/Services/Dashboard/DashboardService.php`)

```php
class DashboardService
{
    public function getStats(): DashboardStats;           // cached 5 min
    public function getActivity(int $limit = 20): Collection;
    public function getUserGrowthChart(string $period): ChartData;
    public function getRevenueChart(string $period): ChartData;
}
```

### AuditService (`app/Services/Audit/AuditService.php`)

```php
class AuditService
{
    public function list(array $filters): LengthAwarePaginator;
    public function show(int $id): Audit;
    public function export(array $filters): StreamedResponse;
    public function log(AuditAction $action, Model $model, ?array $old, ?array $new): void;
}
```

---

## Enums

```php
enum Role: string
{
    case ADMIN = 'admin';
    case EDITOR = 'editor';
    case VIEWER = 'viewer';
}

enum Permission: string
{
    case MANAGE_USERS = 'manage-users';
    case VIEW_USERS = 'view-users';
    case MANAGE_ROLES = 'manage-roles';
    case VIEW_DASHBOARD = 'view-dashboard';
    case VIEW_AUDITS = 'view-audits';
    case EXPORT_AUDITS = 'export-audits';
    case MANAGE_SETTINGS = 'manage-settings';
}

enum AuditAction: string
{
    case CREATED = 'created';
    case UPDATED = 'updated';
    case DELETED = 'deleted';
    case LOGIN = 'login';
    case LOGOUT = 'logout';
    case MFA_VERIFIED = 'mfa_verified';
    case PASSWORD_RESET = 'password_reset';
    case ROLE_CHANGED = 'role_changed';
    case STATUS_CHANGED = 'status_changed';
    case EXPORT = 'export';
}

enum ApiErrorCode: string
{
    case VALIDATION_FAILED = 'VALIDATION_FAILED';
    case UNAUTHORIZED = 'UNAUTHORIZED';
    case FORBIDDEN = 'FORBIDDEN';
    case NOT_FOUND = 'NOT_FOUND';
    case MFA_REQUIRED = 'MFA_REQUIRED';
    case MFA_INVALID = 'MFA_INVALID';
    case TOKEN_EXPIRED = 'TOKEN_EXPIRED';
    case RATE_LIMITED = 'RATE_LIMITED';
    case SERVER_ERROR = 'SERVER_ERROR';
}
```

---

## API Design

### Auth Flow

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

### Response Examples

**Login success (no MFA)**:
```json
{
  "success": true,
  "data": {
    "auth_token": "eyJ...",
    "refresh_token": "eyJ...",
    "expires_in": 3600,
    "user": {
      "id": 1,
      "name": "John Admin",
      "email": "john@example.com",
      "roles": ["admin"],
      "permissions": ["manage-users", "view-dashboard", "view-audits"],
      "mfa_enabled": true
    }
  }
}
```

**Login triggers MFA**:
```json
{
  "success": true,
  "data": {
    "mfa_required": true,
    "mfa_token": "temp_mfa_abc123...",
    "expires_in": 300
  }
}
```

**Paginated response**:
```json
{
  "success": true,
  "data": {
    "items": [],
    "pagination": {
      "current_page": 1,
      "per_page": 15,
      "total": 48,
      "last_page": 4
    }
  }
}
```

---

## Database Schema

```
┌──────────────┐     ┌──────────────────┐     ┌─────────────────┐
│    users      │     │  model_has_roles  │     │      roles       │
├──────────────┤     ├──────────────────┤     ├─────────────────┤
│ id            │◄────│ model_id          │────▶│ id               │
│ name          │     │ role_id           │     │ name             │
│ email         │     └──────────────────┘     │ guard_name       │
│ password      │                               └─────────────────┘
│ mfa_secret    │     ┌──────────────────┐             │
│ mfa_enabled   │     │role_has_permissions│            │
│ status        │     ├──────────────────┤     ┌───────▼─────────┐
│ last_login_at │     │ role_id           │────▶│   permissions    │
│ created_at    │     │ permission_id     │     ├─────────────────┤
│ updated_at    │     └──────────────────┘     │ id               │
│ deleted_at    │                               │ name             │
└──────┬───────┘                               │ guard_name       │
       │                                        └─────────────────┘
       │
       │     ┌───────────────────────────────┐
       └────▶│           audits               │
             ├───────────────────────────────┤
             │ user_id (nullable)             │
             │ action (enum)                  │
             │ auditable_type                 │
             │ auditable_id                   │
             │ old_values (json)              │
             │ new_values (json)              │
             │ ip_address                     │
             │ user_agent                     │
             │ metadata (json)               │
             │ created_at                     │
             └───────────────────────────────┘

┌───────────────────────────────┐
│       notifications            │
├───────────────────────────────┤
│ id                             │
│ user_id → users.id             │
│ type (enum)                    │
│ title                          │
│ body                           │
│ data (json)                    │
│ read_at (nullable)             │
│ created_at                     │
└───────────────────────────────┘

┌───────────────────────────────┐
│     personal_access_tokens     │
├───────────────────────────────┤
│ id                             │
│ tokenable_type                 │
│ tokenable_id → users.id        │
│ name                           │
│ token (hashed)                 │
│ abilities (json)               │
│ last_used_at                   │
│ expires_at                     │
│ created_at                     │
└───────────────────────────────┘
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

### Queue Jobs

| Job | Queue | Triggered By | Purpose |
|-----|-------|-------------|---------|
| `SendOtpEmail` | `otp` | Login MFA fallback | Deliver OTP via email |
| `SendPasswordResetEmail` | `emails` | `forgotPassword()` | Password reset link |
| `SendWelcomeEmail` | `emails` | `UserCreated` event | New user welcome |
| `LogAuditEntry` | `default` | `AuditListener` | Non-blocking audit write |
| `SendDevAlert` | `dev_alerts` | Exception handler | Slack/email for errors |
| `ExportAuditCsv` | `default` | `AuditController@export` | Large export generation |

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

### Do's

- Use `useApiCall` for all HTTP requests (frontend)
- Extend `ApiController` for all controllers (backend)
- Put business logic in Services, not Controllers
- Use Form Requests for validation
- Use Pinia setup stores with `defineStore`
- Declare `authType` and `label` on every route
- Use PHP native enums for constants
- Use Events/Listeners for side effects
- Log all state-changing actions via AuditListener
- Use Policies for resource-level authorization

### Don'ts

- Don't use raw `fetch` or `axios` — use `useApiCall`
- Don't put business logic in controllers
- Don't skip Form Request validation
- Don't use Options API — use Composition API with `<script setup>`
- Don't hardcode constants — use Enums
- Don't bypass authorization — use Policies
- Don't commit without running Pint / ESLint + Prettier
- Don't write raw SQL — use Eloquent with parameterized queries
- Don't log audit entries synchronously — always queue them

---

*This document serves as the single source of truth for our team's technical decisions and coding standards. All new code must adhere to these conventions.*
