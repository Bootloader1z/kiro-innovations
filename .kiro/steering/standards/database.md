---
inclusion: fileMatch
fileMatchPattern: "**/*.{php,sql,migration}"
---

# Database Schema

## Tables

### users
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| name | string | |
| email | string unique | |
| password | string | bcrypt hashed |
| mfa_secret | string nullable | TOTP secret (encrypted at rest) |
| mfa_enabled | boolean | default false |
| status | enum(active,inactive) | |
| last_login_at | timestamp nullable | |
| created_at | timestamp | |
| updated_at | timestamp | |
| deleted_at | timestamp nullable | soft delete |

### audits
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| user_id | bigint FK nullable | null = system action |
| action | enum | see AuditAction enum |
| auditable_type | string | polymorphic model class |
| auditable_id | bigint | polymorphic model ID |
| old_values | json nullable | state before change |
| new_values | json nullable | state after change |
| ip_address | string | |
| user_agent | string | |
| metadata | json nullable | route, request_id, tags |
| created_at | timestamp | |

### notifications
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| user_id | bigint FK | |
| type | enum | notification category |
| title | string | |
| body | text | |
| data | json | deep-link payload |
| read_at | timestamp nullable | null = unread |
| created_at | timestamp | |

### personal_access_tokens (Sanctum)
| Column | Type | Notes |
|---|---|---|
| id | bigint PK | |
| tokenable_type | string | polymorphic |
| tokenable_id | bigint FK | |
| name | string | |
| token | string | SHA-256 hashed |
| abilities | json | |
| last_used_at | timestamp nullable | |
| expires_at | timestamp nullable | |
| created_at | timestamp | |

### roles / permissions (Spatie)
Standard Spatie tables: `roles`, `permissions`, `model_has_roles`, `model_has_permissions`, `role_has_permissions`.

## Relationships

- users ←→ roles (many-to-many via model_has_roles)
- roles ←→ permissions (many-to-many via role_has_permissions)
- users → audits (one-to-many via user_id)
- users → notifications (one-to-many via user_id)
- users → personal_access_tokens (one-to-many via tokenable)
