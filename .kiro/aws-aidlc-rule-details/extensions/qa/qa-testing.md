# Quality Assurance & Testing Framework

## Overview

These QA rules are cross-cutting constraints that define testing standards, quality gates, and QA processes for the AI-DLC workflow. They apply primarily to the **User Stories**, **Functional Design**, **NFR Requirements**, **Test Planning**, **Code Generation**, and **Build and Test** stages.

**Enforcement**: At each applicable stage, the model MUST verify compliance with these rules before presenting the stage completion message to the user.

### Blocking QA Finding Behavior

A **blocking QA finding** means:
1. The finding MUST be listed in the stage completion message under a "QA Findings" section with the QA rule ID and description
2. The stage MUST NOT present the "Continue to Next Stage" option until all blocking findings are resolved
3. The model MUST present only the "Request Changes" option with a clear explanation of what needs to change
4. The finding MUST be logged in `aidlc-docs/audit.md` with the QA rule ID, description, and stage context

If a QA rule is not applicable to the current project (e.g., QA-04 when no state machines or invariants exist), mark it as **N/A** in the compliance summary — this is not a blocking finding.

### Default Enforcement

All rules in this document are **blocking** by default. If any rule's verification criteria are not met, it is a blocking QA finding — follow the blocking finding behavior defined above.

### Partial Enforcement Mode

If **Partial** enforcement is selected (opt-in option B), only the automated quality gates are enforced — i.e., **QA-01** remains blocking. All other rules (QA-02 through QA-07), including the manual QA checklist, become **advisory** (non-blocking). The enforcement mode MUST be logged in `aidlc-docs/aidlc-state.md` under `## Extension Configuration`.

### Verification Criteria Format

Verification items in this document are plain bullet points describing compliance checks. They are distinct from the `- [ ]` / `- [x]` progress-tracking checkboxes used in stage plan files. Each item should be evaluated as compliant or non-compliant during review.

---

## Rule QA-01: Automated Quality Gates

**Rule**: Every pull request MUST pass these gates before merge:

| Gate | Tool | Threshold | Blocking? |
|------|------|-----------|-----------|
| Code Style (PHP) | Laravel Pint (PSR-12) | Zero violations | Yes |
| Code Style (JS/Vue) | ESLint + Prettier | Zero errors (warnings allowed) | Yes |
| Static Analysis | PHPStan (level 6+) | Zero errors | Yes |
| Unit Tests | PHPUnit (PHP) + Vitest (JS) | 100% pass rate | Yes |
| Test Coverage | PHPUnit --coverage | Minimum 80% on business logic | Yes |
| Build | `npm run build` | Successful compilation | Yes |
| Database | `php artisan migrate --pretend` | No migration errors | Yes |

**Verification**:
- All gates listed above pass with zero violations/errors
- Coverage threshold of 80% is met on business logic paths
- No test failures exist in the PR pipeline
- Build completes without errors

---

## Rule QA-02: Testing Strategy & Test Pyramid

**Rule**: The project MUST follow a test pyramid strategy with appropriate distribution of test types.

### Test Pyramid

```
         /  E2E Tests  \          (Few - critical user journeys)
        / Integration    \        (Some - cross-domain interactions)
       /   Unit Tests     \       (Many - business logic, services)
      /  Property-Based    \      (Targeted - invariants, state machines)
```

### Test Types & Ownership

| Test Type | Scope | Written By | Run When |
|-----------|-------|-----------|----------|
| Unit tests | Single class/function | Developer | Every PR |
| Integration tests | Cross-domain interactions | Developer | Every PR |
| Property-based tests | Invariants, state machines | Developer | Every PR |
| E2E tests (browser) | Critical user flows | QA/Developer | Pre-release |
| Performance tests | Response times, throughput | DevOps | Pre-release |
| Accessibility tests | WCAG compliance | QA | Per feature |

**Verification**:
- Test suite includes unit, integration, and property-based tests as appropriate
- Test distribution follows the pyramid (more unit tests than integration, more integration than E2E)
- Each test type has clear ownership and execution triggers defined

---

## Rule QA-03: Test Coverage Requirements

**Rule**: Code coverage MUST meet the following minimum thresholds per area:

| Code Area | Minimum Coverage | Rationale |
|-----------|-----------------|-----------|
| Domain Services | 80% | Core business logic |
| Controllers | 60% | Integration with framework |
| Models | 50% | Relationships, scopes, accessors |
| Middleware | 90% | Security-critical paths |
| Frontend components | 50% | Key interactions and states |

**Verification**:
- Coverage reports confirm each area meets its minimum threshold
- Business-critical paths (Domain Services, Middleware) meet elevated thresholds
- Coverage is measured per area, not just as a global aggregate

---

## Rule QA-04: Property-Based Testing

**Rule**: Property-Based Testing (PBT) is REQUIRED for:
- State machines (ticket status transitions)
- Data transformations (serialization round-trips)
- Business rule invariants (SLA calculations, priority ordering)
- Idempotent operations (deactivation, assignment)

PBT frameworks:
- PHP: Eris (with PHPUnit integration)
- JavaScript: fast-check (with Vitest)

> **Cross-reference**: If the Property-Based Testing extension is enabled (`extensions/testing/property-based/property-based-testing.md`), its PBT-xx rules govern PBT requirements in detail and take precedence. The project's PHP PBT framework is selected per PBT extension rule PBT-09.

**Verification**:
- PBT tests exist for all state machines, data transformations, business rule invariants, and idempotent operations
- PBT frameworks are correctly configured (fast-check for JS; PHP PBT framework per PBT-09 if PBT extension is enabled)
- PBT-discovered failures are captured as permanent example-based regression tests

---

## Rule QA-05: Manual QA Checklist

**Rule**: Before a feature is marked "Done", the following checklist MUST be completed:

- [ ] Acceptance criteria from user story verified
- [ ] Happy path tested manually
- [ ] Error/edge cases tested manually
- [ ] Mobile responsiveness verified (mobile-first)
- [ ] Cross-browser tested (Chrome, Firefox, Safari, Edge)
- [ ] Accessibility spot-check (keyboard navigation, screen reader basics)
- [ ] Performance acceptable (page loads < 2s, API responses < 500ms)
- [ ] No console errors in browser DevTools

**Verification**:
- All checklist items are confirmed complete before feature sign-off
- Evidence of manual testing is documented in the PR or feature ticket

---

## Rule QA-06: Regression Testing Rules

**Rule**: The following regression testing rules MUST be followed:

- All existing tests MUST pass before any PR is merged
- When a bug is fixed, a regression test MUST be added
- PBT-discovered failures MUST be captured as permanent example-based tests
- Test suite MUST run in under 5 minutes (optimize or parallelize if exceeded)
- Flaky tests MUST be investigated immediately, not retried silently

**Verification**:
- No test regressions exist in the PR pipeline
- Bug-fix PRs include corresponding regression tests
- Test suite execution time remains under 5 minutes
- No flaky tests are present without an active investigation ticket

---

## Rule QA-07: Quality Metrics

**Rule**: The following quality metrics MUST be tracked and targets maintained:

| Metric | Target | Measurement |
|--------|--------|-------------|
| Test coverage (business logic) | ≥ 80% | CI pipeline report |
| Build success rate | ≥ 95% | CI pipeline stats |
| Test suite duration | < 5 minutes | CI timing |
| Defect escape rate | < 5% | Bugs in production / total bugs |
| Mean time to fix (critical) | < 4 hours | Issue tracker |

**Verification**:
- Metrics are tracked and visible in CI/CD dashboards
- Targets are met or deviations are documented with remediation plans
- Quality trends are reviewed periodically

---

## Enforcement Integration

| Stage | Applicable Rules | Enforcement |
|-------|-----------------|-------------|
| User Stories | QA | Testable Given/When/Then acceptance criteria required |
| Functional Design | QA-04 | Identify test scenarios and PBT properties |
| NFR Requirements | QA-03, QA-07 | Define coverage and quality metric targets |
| Test Planning | QA-02, QA-04 | Test plan with scenarios and PBT strategy |
| Code Generation | QA-01, QA-04 | Generate tests alongside code; pass automated gates |
| Build and Test | QA-01, QA-06 | Execute all quality gates; run regression suite |
| Operations | — | Placeholder for post-deployment quality monitoring |

At each applicable stage:
- Evaluate all QA rule verification criteria against the artifacts produced
- Include a "QA Compliance" section in the stage completion summary listing each rule as compliant, non-compliant, or N/A
- If any rule is non-compliant, this is a blocking QA finding — follow the blocking finding behavior defined in the Overview
- Before enforcing, check the Enabled status in `aidlc-docs/aidlc-state.md` under `## Extension Configuration`; if the extension is disabled, skip enforcement and log the skip reason
