# Quality Assurance & Testing Framework

**Purpose**: Defines testing standards, quality gates, and QA processes for the AI-DLC workflow.  
**Applies to**: CONSTRUCTION phase (Build and Test stage) and ongoing maintenance.

---

## 1. Automated Quality Gates (CI Pipeline)

Every pull request MUST pass these gates before merge:

| Gate | Tool | Threshold | Blocking? |
|------|------|-----------|-----------|
| Code Style (PHP) | Laravel Pint (PSR-12) | Zero violations | Yes |
| Code Style (JS/Vue) | ESLint + Prettier | Zero errors (warnings allowed) | Yes |
| Static Analysis | PHPStan (level 6+) | Zero errors | Yes |
| Unit Tests | Pest (PHP) + Vitest (JS) | 100% pass rate | Yes |
| Test Coverage | Pest --coverage | Minimum 80% on business logic | Yes |
| Build | `npm run build` | Successful compilation | Yes |
| Database | `php artisan migrate --pretend` | No migration errors | Yes |

---

## 2. Testing Strategy

### 2.1 Test Pyramid

```
         /  E2E Tests  \          (Few - critical user journeys)
        / Integration    \        (Some - cross-domain interactions)
       /   Unit Tests     \       (Many - business logic, services)
      /  Property-Based    \      (Targeted - invariants, state machines)
```

### 2.2 Test Types & Ownership

| Test Type | Scope | Written By | Run When |
|-----------|-------|-----------|----------|
| Unit tests | Single class/function | Developer | Every PR |
| Integration tests | Cross-domain interactions | Developer | Every PR |
| Property-based tests | Invariants, state machines | Developer | Every PR |
| E2E tests (browser) | Critical user flows | QA/Developer | Pre-release |
| Performance tests | Response times, throughput | DevOps | Pre-release |
| Accessibility tests | WCAG compliance | QA | Per feature |

### 2.3 Test Coverage Requirements

| Code Area | Minimum Coverage | Rationale |
|-----------|-----------------|-----------|
| Domain Services | 80% | Core business logic |
| Controllers | 60% | Integration with framework |
| Models | 50% | Relationships, scopes, accessors |
| Middleware | 90% | Security-critical paths |
| Frontend components | 50% | Key interactions and states |

### 2.4 Property-Based Testing (PBT)

PBT is REQUIRED for:
- State machines (ticket status transitions)
- Data transformations (serialization round-trips)
- Business rule invariants (SLA calculations, priority ordering)
- Idempotent operations (deactivation, assignment)

PBT frameworks:
- PHP: Eris (with Pest integration)
- JavaScript: fast-check (with Vitest)

---

## 3. Manual QA Checklist (Per Feature)

Before a feature is marked "Done":

- [ ] Acceptance criteria from user story verified
- [ ] Happy path tested manually
- [ ] Error/edge cases tested manually
- [ ] Mobile responsiveness verified (mobile-first)
- [ ] Cross-browser tested (Chrome, Firefox, Safari, Edge)
- [ ] Accessibility spot-check (keyboard navigation, screen reader basics)
- [ ] Performance acceptable (page loads < 2s, API responses < 500ms)
- [ ] No console errors in browser DevTools

---

## 4. Regression Testing Rules

- All existing tests MUST pass before any PR is merged
- When a bug is fixed, a regression test MUST be added
- PBT-discovered failures MUST be captured as permanent example-based tests
- Test suite MUST run in under 5 minutes (optimize or parallelize if exceeded)
- Flaky tests MUST be investigated immediately, not retried silently

---

## 5. Quality Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Test coverage (business logic) | ≥ 80% | CI pipeline report |
| Build success rate | ≥ 95% | CI pipeline stats |
| Test suite duration | < 5 minutes | CI timing |
| Defect escape rate | < 5% | Bugs in production / total bugs |
| Mean time to fix (critical) | < 4 hours | Issue tracker |

---

## 6. AI-DLC Integration Points

| AI-DLC Phase | QA Activity |
|-------------|-------------|
| User Stories | Write testable acceptance criteria (Given/When/Then) |
| Functional Design | Identify test scenarios and PBT properties |
| NFR Requirements | Define performance/quality targets |
| Code Generation | Generate tests alongside code |
| Build and Test | Execute all quality gates |
| Post-Deployment | Monitor error rates, track defect escapes |
