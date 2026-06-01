# Deployment & CI/CD Framework (GitHub Actions)

**Purpose**: Defines the deployment pipeline, CI/CD configuration, and release management for the AI-DLC workflow.  
**Applies to**: OPERATIONS phase and ongoing delivery.

---

## 1. Pipeline Overview

```
PR Created → Lint → Test → Build → Security Scan → Review → Merge
                                                              ↓
Main Branch → Build → Test → Security → Deploy Staging → Smoke Test → Deploy Production
```

---

## 2. GitHub Actions Workflows

### 2.1 Pull Request Pipeline (`.github/workflows/pr.yml`)

Triggered on every PR to `main`:

```yaml
name: PR Pipeline

on:
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      - run: composer install --no-interaction
      - run: ./vendor/bin/pint --test
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npx eslint resources/js/

  static-analysis:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      - run: composer install --no-interaction
      - run: ./vendor/bin/phpstan analyse --level=6

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
          coverage: xdebug
      - run: composer install --no-interaction
      - run: cp .env.example .env
      - run: php artisan key:generate
      - run: ./vendor/bin/pest --coverage --min=80
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npx vitest run

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build

  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      - run: composer install --no-interaction
      - run: composer audit
      - run: npm ci
      - run: npm audit --audit-level=high
```

### 2.2 Deploy Pipeline (`.github/workflows/deploy.yml`)

Triggered on merge to `main`:

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: shivammathur/setup-php@v2
        with:
          php-version: '8.3'
      - run: composer install --no-interaction --optimize-autoloader --no-dev
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build
      - run: composer install --no-interaction
      - run: cp .env.example .env && php artisan key:generate
      - run: ./vendor/bin/pest

  deploy-staging:
    needs: build-and-test
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Staging
        run: |
          echo "Deploy to staging server"
          # Replace with actual deployment commands:
          # ssh deploy@staging "cd /app && git pull && composer install --no-dev && php artisan migrate --force && npm run build"
      - name: Smoke Test
        run: |
          echo "Run smoke tests against staging"
          # curl -f https://staging.example.com/health || exit 1

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to Production
        run: |
          echo "Deploy to production server"
          # Replace with actual deployment commands:
          # ssh deploy@production "cd /app && git pull && composer install --no-dev && php artisan migrate --force && npm run build && php artisan config:cache && php artisan route:cache"
      - name: Health Check
        run: |
          echo "Verify production health"
          # curl -f https://app.example.com/health || exit 1
      - name: Notify
        if: success()
        run: echo "Deployment successful"
```

---

## 3. Environment Strategy

| Environment | Purpose | Deploy Trigger | URL |
|-------------|---------|---------------|-----|
| Local | Development | Manual (docker compose) | http://localhost:8000 |
| Staging | Pre-production testing | Auto on merge to main | https://staging.example.com |
| Production | Live users | Manual approval after staging | https://app.example.com |

### Environment Variables

| Variable | Local | Staging | Production |
|----------|-------|---------|-----------|
| APP_ENV | local | staging | production |
| APP_DEBUG | true | false | false |
| DB_CONNECTION | sqlite | mysql | mysql |
| CACHE_DRIVER | array | redis | redis |
| SESSION_DRIVER | file | redis | redis |
| LOG_LEVEL | debug | info | info |

---

## 4. Deployment Checklist

### Pre-Deployment
- [ ] All CI gates pass (lint, test, build, security)
- [ ] Code review approved
- [ ] Database migrations tested (`migrate --pretend`)
- [ ] No breaking changes (or migration path documented)
- [ ] Environment variables updated (if new ones added)
- [ ] Rollback plan documented

### Deployment Steps
1. Pull latest code
2. Install dependencies (`composer install --no-dev`)
3. Run migrations (`php artisan migrate --force`)
4. Build frontend (`npm run build`)
5. Clear and rebuild caches (`config:cache`, `route:cache`, `view:cache`)
6. Restart queue workers (if applicable)
7. Verify health check endpoint

### Post-Deployment
- [ ] Health check passes (`/health` returns 200)
- [ ] Smoke test critical paths (login, create ticket, view dashboard)
- [ ] Monitor error rates for 15 minutes
- [ ] Check application logs for unexpected errors
- [ ] Verify scheduled jobs running (SLA escalation check)

---

## 5. Rollback Strategy

### When to Rollback
- Health check fails after deployment
- Error rate spikes above 5% within 15 minutes
- Critical functionality broken (login, ticket creation)
- Data corruption detected

### Rollback Steps
1. Revert to previous release (`git revert` or redeploy previous tag)
2. Run down migrations if needed (`php artisan migrate:rollback --step=1`)
3. Rebuild caches
4. Verify health check
5. Notify team of rollback and root cause

### Rollback SLA
- Decision to rollback: within 15 minutes of detecting issue
- Rollback execution: within 30 minutes
- Post-mortem: within 24 hours

---

## 6. Release Management

### Versioning
- Follow Semantic Versioning (SemVer): `MAJOR.MINOR.PATCH`
- Tag releases in Git: `v1.0.0`, `v1.1.0`, `v1.1.1`
- Maintain CHANGELOG.md with notable changes per version

### Release Cadence
- **Regular releases**: Weekly (or bi-weekly)
- **Hotfixes**: As needed (bypass normal cycle for critical fixes)
- **Feature flags**: Use for gradual rollout of major features

### Release Notes Template
```markdown
## v1.x.x (YYYY-MM-DD)

### Added
- [Feature description]

### Changed
- [Change description]

### Fixed
- [Bug fix description]

### Security
- [Security patch description]
```

---

## 7. Monitoring & Alerting (Post-Deploy)

### Health Monitoring
| Check | Frequency | Alert Threshold |
|-------|-----------|-----------------|
| `/health` endpoint | Every 30 seconds | 2 consecutive failures |
| Error rate | Continuous | > 5% of requests |
| Response time (p95) | Continuous | > 2 seconds |
| Queue depth | Every minute | > 100 pending jobs |
| Disk space | Every 5 minutes | < 10% free |

### Alert Channels
| Severity | Channel | Response |
|----------|---------|----------|
| Critical | PagerDuty / SMS | Immediate response |
| High | Slack #alerts | Response within 1 hour |
| Medium | Slack #monitoring | Response within 4 hours |
| Low | Email digest | Review next business day |

---

## 8. AI-DLC Integration Points

| AI-DLC Phase | Deployment Activity |
|-------------|---------------------|
| Code Generation | Generate Dockerfile, docker-compose, CI configs |
| Build and Test | Validate CI pipeline passes |
| Operations | Deploy, monitor, maintain |

### DORA Metrics to Track

| Metric | Target (Elite) | How to Measure |
|--------|---------------|----------------|
| Deployment Frequency | Multiple times/week | Count deploys to production |
| Lead Time for Changes | < 1 day | PR merge → production deploy time |
| Mean Time to Restore | < 1 hour | Incident start → resolution time |
| Change Failure Rate | < 15% | Rollbacks / total deploys |
