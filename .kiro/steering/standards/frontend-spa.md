---
inclusion: fileMatch
fileMatchPattern: "**/*.{vue,ts,tsx}"
---

# Frontend SPA — webkit-spa-prime

## Directory Structure

```
src/
├── components/
│   ├── auth-page/         LoginForm, MfaPrompt, ForgotPassword, ResetPassword
│   ├── dashboard/         StatsCard, StatsGrid, ActivityFeed, RevenueChart, UserGrowthChart
│   ├── users/             UserTable, UserCreateDialog, UserEditDialog, UserRoleAssign, UserActionMenu
│   ├── notifications/     NotificationBell, NotificationPanel, NotificationItem
│   ├── settings/          ProfileForm, PasswordChange, MfaSetup, ThemeToggle
│   ├── layout/            AppTopbar, AppSidebar, AppFooter, AppLayout
│   └── webkit/            WbInputText, WbPassword, WbDropdown, WbDataTable, WbDialog, WbButton, WbToast, WbConfirm
├── composables/           network.ts, theme.ts, sidebar.ts, permissions.ts, pagination.ts, toast.ts, debounce.ts
├── stores/                auth.store.ts, user.store.ts, notification.store.ts, settings.store.ts
├── typings/               auth.types.ts, user.types.ts, notification.types.ts, api.types.ts, common.types.ts
├── views/                 LoginPage, DashboardPage, UsersPage, NotificationsPage, SettingsPage, NotFoundPage
└── router/index.ts        Route definitions + navigation guards
```

## UI/UX Layout

```
┌─────────────────────────────────────────────────────────────┐
│  TOPBAR: [Menu] [Search...] [Bell] [Theme] [Avatar]          │
├─────────┬───────────────────────────────────────────────────┤
│ SIDEBAR │  CONTENT: Stats cards → Charts + Activity Feed     │
│ (collap)│                                                    │
└─────────┴───────────────────────────────────────────────────┘
```

## Design Tokens

| Token | Light | Dark |
|---|---|---|
| Background | `#FFFFFF` | `#1E1E2E` |
| Surface | `#F8FAFC` | `#2A2A3C` |
| Primary | `#3B82F6` | `#60A5FA` |
| Sidebar BG | `#1E293B` | `#0F172A` |
| Text | `#1E293B` | `#E2E8F0` |
| Border | `#E2E8F0` | `#334155` |
| Success | `#10B981` | `#34D399` |
| Danger | `#EF4444` | `#F87171` |

## UX Patterns

- Sidebar: full desktop, overlay mobile, collapsible to icons
- Toast: bottom-right, auto-dismiss 5s, stacked
- Loading: PrimeVue Skeleton on all async data
- Optimistic UI: status toggles instant, revert on failure
- Shortcuts: `/` focus search, `Esc` close modals
- Breadcrumbs: from `route.meta.label`
- Empty states: illustrated placeholder

## Route Guards

```typescript
interface RouteMeta {
  authType: 'public' | 'authenticated' | 'mfa-verified'
  label: string
  roles?: string[]
  icon?: string
}
// 1. public → pass | 2. no token → /login | 3. mfa pending → /mfa | 4. roles check → /403
```

## Conventions

| Rule | Convention |
|---|---|
| File naming | PascalCase (`LoginForm.vue`) |
| Folders | Feature-scoped (`components/auth-page/`) |
| SFC order | `<script>` → `<template>` → `<style>` |
| Path alias | `@/` = `src/` |
| API calls | Always `useApiCall` — never raw fetch/axios |
| Stores | `{domain}.store.ts` with setup syntax |
| Composables | `use{Name}` pattern |
| Types | `{domain}.types.ts` in `src/typings/` |
| Session | `useStorage` from VueUse for auth hydration |
