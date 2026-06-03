---
inclusion: fileMatch
fileMatchPattern: "**/*.php"
---

# Backend API — webkit-api

## Directory Structure

```
app/
├── Auth/          ApiKeyGuard, MultiTokenGuard
├── Enums/         Role, Permission, AuditAction, ApiErrorCode
├── Events/        UserCreated, UserUpdated, UserDeleted, UserLoggedIn, UserLoggedOut, MfaVerified, PasswordReset, RoleAssigned, StatusToggled, AuditExported
├── Facades/       ConversionHelper, PaginationHelper
├── Http/
│   ├── Controllers/   Thin, extend ApiController
│   ├── Middleware/     MFA gate, ForceJsonResponse, LogRequestId
│   └── Requests/      All validation in *Request classes
├── Listeners/     AuditListener (queues LogAuditEntry)
├── Models/        Eloquent models only — no business logic
├── Policies/      Resource-level authorization
├── QueryFilters/  Reusable filter classes
├── Services/
│   ├── Auth/      AuthService, MfaService
│   ├── User/      UserService
│   ├── Notification/ NotificationService
│   ├── Dashboard/ DashboardService
│   └── Audit/     AuditService
└── Traits/        Controllers/, Services/
```

## API Routes (`/api/v1/`)

### Auth (`auth.routes.php`)
| Method | Endpoint | Middleware |
|---|---|---|
| POST | `/auth/login` | `throttle:5,1` |
| POST | `/auth/mfa/verify` | `throttle:3,1` |
| POST | `/auth/refresh` | `auth:sanctum` |
| POST | `/auth/logout` | `auth:sanctum` |
| POST | `/auth/forgot-password` | `throttle:3,5` |
| POST | `/auth/reset-password` | `throttle:3,5` |

### Users (`user.routes.php`)
| Method | Endpoint | Middleware |
|---|---|---|
| GET | `/users` | `auth:sanctum`, `role:admin` |
| POST | `/users` | `auth:sanctum`, `role:admin` |
| GET | `/users/{id}` | `auth:sanctum`, `role:admin` |
| PUT | `/users/{id}` | `auth:sanctum`, `role:admin` |
| DELETE | `/users/{id}` | `auth:sanctum`, `role:admin` |
| PATCH | `/users/{id}/status` | `auth:sanctum`, `role:admin` |
| POST | `/users/{id}/roles` | `auth:sanctum`, `role:admin` |
| DELETE | `/users/{id}/roles` | `auth:sanctum`, `role:admin` |
| POST | `/users/{id}/reset-password` | `auth:sanctum`, `role:admin` |

### Profile (`profile.routes.php`)
| Method | Endpoint | Middleware |
|---|---|---|
| GET | `/profile` | `auth:sanctum` |
| PUT | `/profile` | `auth:sanctum` |
| PUT | `/profile/password` | `auth:sanctum` |
| POST | `/profile/mfa/enable` | `auth:sanctum` |
| POST | `/profile/mfa/confirm` | `auth:sanctum` |
| DELETE | `/profile/mfa/disable` | `auth:sanctum` |

### Notifications (`notification.routes.php`)
| Method | Endpoint | Middleware |
|---|---|---|
| GET | `/notifications` | `auth:sanctum` |
| GET | `/notifications/unread-count` | `auth:sanctum` |
| PATCH | `/notifications/{id}/read` | `auth:sanctum` |
| PATCH | `/notifications/read-all` | `auth:sanctum` |

### Dashboard (`dashboard.routes.php`)
| Method | Endpoint | Middleware |
|---|---|---|
| GET | `/dashboard/stats` | `auth:sanctum` |
| GET | `/dashboard/activity` | `auth:sanctum` |
| GET | `/dashboard/charts/users` | `auth:sanctum`, `role:admin` |
| GET | `/dashboard/charts/revenue` | `auth:sanctum`, `role:admin` |

### Audits (`audit.routes.php`)
| Method | Endpoint | Middleware |
|---|---|---|
| GET | `/audits` | `auth:sanctum`, `role:admin` |
| GET | `/audits/{id}` | `auth:sanctum`, `role:admin` |
| GET | `/audits/export` | `auth:sanctum`, `role:admin` |

### Roles (`role.routes.php`)
| Method | Endpoint | Middleware |
|---|---|---|
| GET | `/roles` | `auth:sanctum`, `role:admin` |
| POST | `/roles` | `auth:sanctum`, `role:admin` |
| PUT | `/roles/{id}` | `auth:sanctum`, `role:admin` |
| DELETE | `/roles/{id}` | `auth:sanctum`, `role:admin` |
| GET | `/permissions` | `auth:sanctum`, `role:admin` |

## Security Pipeline (execution order)

1. Rate Limiting → 2. CORS → 3. ForceJsonResponse → 4. LogRequestId → 5. Sanctum Auth → 6. MFA Gate → 7. Role/Permission → 8. Form Request → 9. Policy → 10. Controller → 11. Audit (queued)

## Core Services

```php
// AuthService: login, verifyMfa, refreshToken, logout, forgotPassword, resetPassword
// MfaService: generateSecret, confirmSetup, verify, disable, generateRecoveryCodes
// UserService: list, create, update, delete, toggleStatus, assignRoles, revokeRoles, forceResetPassword
// NotificationService: list, unreadCount, markRead, markAllRead, send
// DashboardService: getStats (cached 5m), getActivity, getUserGrowthChart, getRevenueChart
// AuditService: list, show, export, log
```

## Enums

```php
enum Role: string { case ADMIN='admin'; case EDITOR='editor'; case VIEWER='viewer'; }
enum Permission: string { case MANAGE_USERS='manage-users'; case VIEW_USERS='view-users'; case MANAGE_ROLES='manage-roles'; case VIEW_DASHBOARD='view-dashboard'; case VIEW_AUDITS='view-audits'; case EXPORT_AUDITS='export-audits'; case MANAGE_SETTINGS='manage-settings'; }
enum AuditAction: string { case CREATED='created'; case UPDATED='updated'; case DELETED='deleted'; case LOGIN='login'; case LOGOUT='logout'; case MFA_VERIFIED='mfa_verified'; case PASSWORD_RESET='password_reset'; case ROLE_CHANGED='role_changed'; case STATUS_CHANGED='status_changed'; case EXPORT='export'; }
enum ApiErrorCode: string { case VALIDATION_FAILED='VALIDATION_FAILED'; case UNAUTHORIZED='UNAUTHORIZED'; case FORBIDDEN='FORBIDDEN'; case NOT_FOUND='NOT_FOUND'; case MFA_REQUIRED='MFA_REQUIRED'; case MFA_INVALID='MFA_INVALID'; case TOKEN_EXPIRED='TOKEN_EXPIRED'; case RATE_LIMITED='RATE_LIMITED'; case SERVER_ERROR='SERVER_ERROR'; }
```

## Response Shapes

```json
{"success": true, "data": {...}}
{"success": false, "error_code": "...", "error_message": "..."}
{"success": true, "data": {"items": [], "pagination": {"current_page":1,"per_page":15,"total":48,"last_page":4}}}
```
