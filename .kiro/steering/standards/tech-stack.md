---
inclusion: fileMatch
fileMatchPattern: "**/*.{vue,ts,php,json}"
---

# Tech Stack

## Frontend — `webkit-spa-prime`

| Category | Technology | Details |
|---|---|---|
| Framework | Vue 3 | Composition API + `<script setup>` |
| Language | TypeScript | Strict mode |
| Build | Vite 5 | HMR, optimized builds |
| CSS | Tailwind CSS 3.4 | JIT, dark mode via `class` |
| UI | PrimeVue 3.46 | Custom Tailwind preset |
| State | Pinia | Setup stores `defineStore()` |
| Routing | Vue Router 4 | History mode, auth/role/MFA guards |
| HTTP | VueUse `useFetch` | Wrapped via `useApiCall` composable |
| Validation | Vuelidate | Template + model-based |
| Icons | Font Awesome | `@fortawesome/vue-fontawesome` |
| Charts | ApexCharts | `vue3-apexcharts` |
| Lint | ESLint + Prettier | Single quotes, no semi, 130 width, 2-space |
| Hooks | Husky | Pre-commit lint |

## Backend — `webkit-api`

| Category | Technology | Details |
|---|---|---|
| Framework | Laravel 10 | MVC + Service layer |
| Language | PHP 8.2+ | Native enums, typed properties |
| Auth | Sanctum + JWT | `stechstudio/laravel-jwt` |
| Permissions | Spatie Permission 6 | Role + permission-based |
| MFA | Google2FA + OTPHP | TOTP (RFC 6238) |
| Queue | Redis (predis) | `otp`, `emails`, `dev_alerts`, `default` |
| Storage | AWS S3 | `league/flysystem-aws-s3-v3` |
| Style | Laravel Pint | PSR-12 |
| Analysis | GrumPHP | Pint + Enlightn + PHPUnit |
| Testing | PHPUnit 10 | Unit + feature |
| Containers | Docker Compose | app, queue-worker, scheduler, MySQL, Redis |
| CI/CD | GitLab CI | `.gitlab-ci.yml` |
| Versioning | URI prefix | `/api/v1/` |
