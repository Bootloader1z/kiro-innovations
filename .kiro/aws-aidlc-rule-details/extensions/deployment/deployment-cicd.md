# Deployment & CI/CD Framework

## Overview

These deployment and CI/CD rules are cross-cutting constraints that apply primarily to the **Infrastructure Design**, **Code Generation**, **Build and Test**, and **Operations** (placeholder) phases of the AI-DLC workflow. They define the required pipeline gates, environment strategy, deployment procedures, rollback plans, release management, and monitoring standards that all projects MUST follow.

**Enforcement**: At each applicable stage, the model MUST verify compliance with these rules before presenting the stage completion message to the user.

### Blocking Deployment Finding Behavior

A **blocking deployment finding** means:
1. The finding MUST be listed in the stage completion message under a "Deployment Findings" section with the DEPLOY rule ID and description
2. The stage MUST NOT present the "Continue to Next Stage" option until all blocking findings are resolved
3. The model MUST present only the "Request Changes" option with a clear explanation of what needs to change
4. The finding MUST be logged in `aidlc-docs/audit.md` with the DEPLOY rule ID, description, and stage context

If a DEPLOY rule is not applicable to the current project (e.g., DEPLOY-06 when no production environment exists yet), mark it as **N/A** in the compliance summary — this is not a blocking finding.

### Default Enforcement

All rules in this document are **blocking** by default. If any rule's verification criteria are not met, it is a blocking deployment finding — follow the blocking finding behavior defined above.

### Partial Enforcement Mode

If **Partial** (option B) is selected during opt-in, only the CI pipeline gate rules are enforced:
- **Enforced (blocking)**: DEPLOY-01 — lint, test, build, and security-scan gates
- **Advisory (non-blocking)**: DEPLOY-02 through DEPLOY-07 — environment strategy, deployment automation, rollback, release management, monitoring, and DORA metrics

In Partial mode, advisory rules are evaluated and reported but do NOT block stage progression. Log the enforcement mode in `aidlc-docs/aidlc-state.md` under `## Extension Configuration`.

### Verification Criteria Format

Verification items in this document are plain bullet points describing compliance checks. They are distinct from the `- [ ]` / `- [x]` progress-tracking checkboxes used in stage plan files. Each item should be evaluated as compliant or non-compliant during review.

---

## Rule DEPLOY-01: CI Pipeline Gates

**Rule**: The merge/pull request pipeline MUST include the following gates, all configured as blocking (pipeline fails if any gate fails). Rules are CI-provider-agnostic — they define WHAT gates must exist, not which vendor runs them.

Required gates:
1. **Lint** — code style enforcement (e.g., Laravel Pint for PHP, ESLint/Prettier for JS/TS)
2. **Static Analysis** — automated code quality and security analysis (e.g., GrumPHP with Enlightn Security Checker)
3. **Test + Coverage** — automated test suite with minimum ~80% line coverage threshold; pipeline fails if coverage drops below threshold
4. **Build** — application builds successfully (frontend assets, container image, etc.)
5. **Dependency / Security Scan** — `composer audit`, `npm audit`, or equivalent; fails on high/critical vulnerabilities

All gates MUST be blocking — a failure in any gate prevents merge.

**Verification**:
- CI configuration file defines all five gate stages
- All gates are configured to fail the pipeline on error (no `allow_failure` or equivalent on required gates)
- Coverage threshold is configured at ≥80%
- Dependency audit step fails on high or critical severity vulnerabilities
- No gate is skippable without explicit documented exception

**Illustrative Example** (GitLab CI — see `team-standards.md` for the team's actual tooling):

```yaml
# .gitlab-ci.yml (simplified — refer to team-standards.md for canonical config)
stages: [lint, analyse, test, build, security]

lint:
  stage: lint
  script:
    - composer install --no-interaction
    - ./vendor/bin/pint --test

analyse:
  stage: analyse
  script:
    - ./vendor/bin/grumphp run

test:
  stage: test
  script:
    - cp .env.example .env && php artisan key:generate
    - ./vendor/bin/phpunit --coverage-text --coverage-clover=coverage.xml
  coverage: '/Lines:\s+(\d+\.\d+)%/'

build:
  stage: build
  script:
    - docker build -t $ECR_REPO:$CI_COMMIT_SHA .

security:
  stage: security
  script:
    - composer audit
    - npm audit --audit-level=high
```

---

## Rule DEPLOY-02: Environment Strategy

**Rule**: The project MUST define a minimum of three environments with clear separation of purpose, deploy triggers, and configuration.

| Environment | Purpose | Deploy Trigger | URL Pattern |
|-------------|---------|---------------|-------------|
| Local | Development & debugging | Manual (`docker compose up`) | `http://localhost:8000` |
| Staging | Pre-production validation | Auto on merge to main branch | `https://staging.example.com` |
| Production | Live users | Manual approval after staging passes | `https://app.example.com` |

### Environment Variable Matrix

| Variable | Local | Staging | Production |
|----------|-------|---------|-----------|
| `APP_ENV` | `local` | `staging` | `production` |
| `APP_DEBUG` | `true` | `false` | `false` |
| `DB_CONNECTION` | `mysql` | `mysql` | `mysql` |
| `CACHE_DRIVER` | `array` | `redis` | `redis` |
| `SESSION_DRIVER` | `file` | `redis` | `redis` |
| `QUEUE_CONNECTION` | `sync` | `redis` | `redis` |
| `LOG_LEVEL` | `debug` | `info` | `info` |

**Verification**:
- At least three environments are defined (local, staging, production)
- Each environment has a distinct `APP_ENV` value
- Production never has `APP_DEBUG=true`
- Environment-specific configuration is managed via environment variables (not hardcoded)
- Database and cache drivers are consistent with the team's infrastructure (MySQL 8, Redis)

---

## Rule DEPLOY-03: Deployment Checklist

**Rule**: Every deployment MUST follow a documented checklist covering pre-deployment validation, deployment execution, and post-deployment verification.

### Pre-Deployment
- All CI gates pass (lint, test, build, security)
- Code review approved and merged
- Database migrations tested (`php artisan migrate --pretend`)
- No breaking changes (or migration path documented)
- Environment variables updated if new ones are introduced
- Rollback plan documented

### Deployment Steps
1. Pull latest code / deploy new container image
2. Install dependencies (`composer install --no-dev --optimize-autoloader`)
3. Run migrations (`php artisan migrate --force`)
4. Build frontend assets (`npm run build`) or deploy pre-built assets
5. Clear and rebuild caches (`config:cache`, `route:cache`, `view:cache`)
6. Restart queue workers (if applicable)
7. Verify health check endpoint responds

### Post-Deployment
- Health check passes (`/health` returns 200)
- Smoke test critical paths (login, core workflows)
- Monitor error rates for 15 minutes
- Check application logs for unexpected errors
- Verify scheduled jobs and queue workers are running

**Verification**:
- A deployment checklist document or runbook exists
- Pre-deployment validation steps are documented and followed
- Post-deployment verification steps are documented and followed
- Health check endpoint exists and is verified after each deployment

---

## Rule DEPLOY-04: Rollback Strategy

**Rule**: Every deployment MUST have a documented rollback strategy with clear triggers, steps, and time-bound SLAs.

### When to Rollback
- Health check fails after deployment
- Error rate spikes above 5% within 15 minutes
- Critical functionality broken (authentication, core business flows)
- Data corruption detected

### Rollback Steps
1. Revert to previous release (redeploy previous container image/tag or `git revert`)
2. Run down migrations if needed (`php artisan migrate:rollback --step=1`)
3. Rebuild caches
4. Verify health check passes
5. Notify team of rollback and begin root cause analysis

### Rollback SLA
| Action | Target |
|--------|--------|
| Decision to rollback | Within 15 minutes of detecting issue |
| Rollback execution | Within 30 minutes |
| Post-mortem documented | Within 24 hours |

**Verification**:
- Rollback procedure is documented
- Previous deployment artifacts (images/tags) are retained and accessible
- Rollback can be executed without re-running the full CI pipeline
- Rollback SLA targets are defined

---

## Rule DEPLOY-05: Release Management

**Rule**: The project MUST follow a structured release management process with semantic versioning, defined cadence, and documented changes.

### Versioning
- Follow Semantic Versioning (SemVer): `MAJOR.MINOR.PATCH`
- Tag releases in Git: `v1.0.0`, `v1.1.0`, `v1.1.1`
- Maintain `CHANGELOG.md` with notable changes per version

### Release Cadence
| Type | Frequency | Notes |
|------|-----------|-------|
| Regular releases | Weekly or bi-weekly | Planned feature batches |
| Hotfixes | As needed | Bypass normal cycle for critical fixes |
| Feature flags | As needed | Gradual rollout of major features |

### Release Notes Template

```markdown
## vX.Y.Z (YYYY-MM-DD)

### Added
- [Feature description]

### Changed
- [Change description]

### Fixed
- [Bug fix description]

### Security
- [Security patch description]
```

**Verification**:
- SemVer is used for all release tags
- `CHANGELOG.md` exists and is updated with each release
- Release notes follow a consistent format
- Git tags correspond to deployed versions

---

## Rule DEPLOY-06: Monitoring & Alerting

**Rule**: Production deployments MUST have health monitoring and alerting configured with defined thresholds and escalation paths.

### Health Monitoring

| Check | Frequency | Alert Threshold |
|-------|-----------|-----------------|
| `/health` endpoint | Every 30 seconds | 2 consecutive failures |
| Error rate | Continuous | > 5% of requests |
| Response time (p95) | Continuous | > 2 seconds |
| Queue depth | Every minute | > 100 pending jobs |
| Disk space | Every 5 minutes | < 10% free |

### Alert Channels & Severity

| Severity | Channel | Response Time |
|----------|---------|---------------|
| Critical | PagerDuty / SMS | Immediate |
| High | Slack #alerts | Within 1 hour |
| Medium | Slack #monitoring | Within 4 hours |
| Low | Email digest | Next business day |

**Verification**:
- Health check endpoint exists and is monitored
- Alert thresholds are defined for error rate, response time, and resource usage
- Alert routing is configured with severity-based escalation
- On-call rotation or response ownership is defined

---

## Rule DEPLOY-07: DORA Metrics

**Rule**: The team MUST track the four DORA metrics with defined targets to measure delivery performance.

| Metric | Target (Elite) | How to Measure |
|--------|---------------|----------------|
| Deployment Frequency | Multiple times per week | Count deploys to production |
| Lead Time for Changes | < 1 day | Time from merge to production deploy |
| Mean Time to Restore (MTTR) | < 1 hour | Incident start → resolution time |
| Change Failure Rate | < 15% | Rollbacks ÷ total deploys |

**Verification**:
- DORA metrics are tracked (manually or via tooling)
- Targets are defined and reviewed periodically
- Change failure rate is calculated from rollback/hotfix data
- Lead time is measured from merge to production availability

---

## Enforcement Integration

| Stage | Applicable Rules | Enforcement |
|-------|-----------------|-------------|
| Infrastructure Design | DEPLOY-02 | Define environments, infrastructure configuration, and env-var matrix |
| Code Generation | DEPLOY-01, DEPLOY-05 | Generate CI pipeline config, Dockerfile, versioning setup |
| Build and Test | DEPLOY-01 | Validate all pipeline gates pass before merge |
| Operations (placeholder) | DEPLOY-03, DEPLOY-04, DEPLOY-06, DEPLOY-07 | Execute deployment checklist, rollback procedures, monitoring, and DORA tracking |

> **Note**: The Operations phase is currently a placeholder in this AI-DLC workflow. DEPLOY rules tied to Operations are enforced when that work is actually performed (e.g., during a real deployment event).

At each applicable stage:
- Evaluate all DEPLOY rule verification criteria against the artifacts produced
- Include a "Deployment Compliance" section in the stage completion summary listing each rule as compliant, non-compliant, or N/A
- If any rule is non-compliant, this is a blocking deployment finding — follow the blocking finding behavior defined in the Overview
- Before evaluating, check the Enabled status in `aidlc-docs/aidlc-state.md` under `## Extension Configuration`
- If the extension is disabled, skip enforcement and log "Deployment & CI/CD extension disabled — skipped" in the stage completion summary
