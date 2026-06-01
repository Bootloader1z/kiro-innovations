# Security Compliance Framework

**Purpose**: Defines security standards, compliance checklists, and vulnerability management for the AI-DLC workflow.  
**Applies to**: All phases (cross-cutting concern). Enforced as blocking constraints.

---

## 1. Security Review Triggers

A dedicated security review is REQUIRED when changes touch:

- Authentication or session management
- Authorization logic (roles, permissions, policies)
- User input handling (new endpoints, form changes)
- Database queries (especially raw/custom queries)
- File upload or download functionality
- External API integrations
- Cryptographic operations
- Environment variables or secrets management
- Docker/deployment configuration
- Dependencies (new packages added)

---

## 2. Security Compliance Checklist (Per Release)

### Authentication & Session (OWASP A07)
- [ ] Passwords hashed with adaptive algorithm (bcrypt cost 12+)
- [ ] Account lockout after failed attempts (5 attempts, 15-min lockout)
- [ ] Session timeout configured (2h idle, 8h absolute)
- [ ] Session invalidated on logout
- [ ] Cookies: Secure, HttpOnly, SameSite=Lax
- [ ] No credentials in source code or logs
- [ ] MFA supported for admin accounts (when applicable)

### Access Control (OWASP A01)
- [ ] All routes require authentication (except explicit public routes)
- [ ] RBAC enforced server-side on every endpoint
- [ ] Object-level authorization (IDOR prevention — users access only their own resources)
- [ ] Admin functions have explicit role checks
- [ ] CORS restricted to allowed origins (no wildcard on authenticated endpoints)
- [ ] Rate limiting on public-facing endpoints

### Input Validation & Injection Prevention (OWASP A03)
- [ ] All inputs validated server-side (Form Requests / validation schemas)
- [ ] String inputs have max-length constraints
- [ ] File uploads validated (MIME type allowlist, size limits, no executables)
- [ ] No raw user input in SQL queries (parameterized queries only)
- [ ] No raw user input in shell commands or file paths
- [ ] Request body size limits configured
- [ ] HTML/script content escaped or rejected (XSS prevention)

### Security Headers
- [ ] Content-Security-Policy: `default-src 'self'` (minimum)
- [ ] Strict-Transport-Security: `max-age=31536000; includeSubDomains`
- [ ] X-Content-Type-Options: `nosniff`
- [ ] X-Frame-Options: `DENY`
- [ ] Referrer-Policy: `strict-origin-when-cross-origin`

### Error Handling & Logging (OWASP A09)
- [ ] Production errors return generic messages only
- [ ] No stack traces, SQL queries, or file paths exposed to users
- [ ] Global exception handler configured
- [ ] Security events logged: failed logins, access denied, privilege changes
- [ ] Logs include: timestamp, request_id, user_id, action
- [ ] Logs do NOT contain: passwords, tokens, session IDs, PII
- [ ] Log retention: minimum 90 days

### Supply Chain Security (OWASP A06)
- [ ] Lock files committed (composer.lock, package-lock.json)
- [ ] No unused dependencies in project
- [ ] `composer audit` passes (zero critical/high)
- [ ] `npm audit` passes (zero critical/high)
- [ ] Docker images use pinned versions (no `:latest` in production)
- [ ] Dependencies sourced from official registries only

### Data Protection
- [ ] Sensitive data encrypted at rest (database encryption enabled)
- [ ] All data in transit over TLS 1.2+
- [ ] PII minimized (only collect what's needed)
- [ ] Data retention policies defined and enforced
- [ ] Audit trail for critical data changes (who, what, when)
- [ ] Backups encrypted and access-controlled

### Secure Design Principles
- [ ] Security logic isolated in dedicated modules (not scattered)
- [ ] Defense in depth (multiple layers: validation + auth + encryption)
- [ ] Fail closed (errors deny access, never fail open)
- [ ] Least privilege (roles have minimum necessary permissions)
- [ ] Misuse cases considered in design (not just happy path)

---

## 3. Security Scanning Schedule

| Scan Type | Frequency | Tool | Owner |
|-----------|-----------|------|-------|
| Dependency vulnerabilities | Every PR + weekly | `composer audit`, `npm audit` | CI Pipeline |
| Static analysis (security) | Every PR | PHPStan (security rules) | CI Pipeline |
| OWASP Top 10 review | Per release | Manual checklist (Section 2) | Security reviewer |
| Dynamic application testing | Monthly | OWASP ZAP | Security team |
| Penetration testing | Quarterly | External firm or OWASP ZAP | Security team |
| Infrastructure review | Per deployment change | Manual review | DevOps |
| Secret scanning | Every PR | GitHub secret scanning / gitleaks | CI Pipeline |

---

## 4. Vulnerability Response SLA

| Severity | Response Time | Patch Deadline | Action |
|----------|--------------|----------------|--------|
| Critical (actively exploited) | Immediate | Hours | Hotfix + emergency deploy |
| High (exploitable, no known exploit) | 24 hours | 1 week | Priority patch |
| Medium (requires specific conditions) | 1 week | 1 month | Scheduled patch |
| Low (theoretical, defense-in-depth) | 1 month | Next release | Normal workflow |

### Vulnerability Handling Process

1. **Detect**: Automated scan or manual report
2. **Triage**: Assess severity and exploitability
3. **Assign**: Route to responsible developer
4. **Fix**: Develop and test patch
5. **Verify**: Security reviewer confirms fix
6. **Deploy**: Push to production within SLA
7. **Document**: Record in security log with root cause

---

## 5. Security Incident Response

### Incident Classification

| Level | Description | Example |
|-------|-------------|---------|
| P1 | Active breach, data exposed | Unauthorized data access |
| P2 | Vulnerability being exploited | Brute-force attack succeeding |
| P3 | Vulnerability discovered, not exploited | Dependency CVE published |
| P4 | Security improvement needed | Missing header, weak config |

### Response Actions

| Level | Immediate Action | Communication |
|-------|-----------------|---------------|
| P1 | Isolate affected systems, preserve evidence | Notify stakeholders within 1 hour |
| P2 | Block attack vector, deploy mitigation | Notify team lead within 4 hours |
| P3 | Assess impact, schedule fix | Track in issue tracker |
| P4 | Add to backlog | Address in next sprint |

---

## 6. AI-DLC Integration Points

| AI-DLC Phase | Security Activity |
|-------------|-------------------|
| Requirements Analysis | Identify security requirements, data sensitivity |
| User Stories | Include misuse cases and abuse scenarios |
| Functional Design | Threat modeling (identify data flows and trust boundaries) |
| NFR Requirements | Define security controls and compliance targets |
| NFR Design | Design security patterns (auth, RBAC, encryption) |
| Code Generation | Apply secure coding patterns, generate security tests |
| Build and Test | Run security scans (SAST, dependency audit) |
| Code Review | Security review for sensitive changes |
| Pre-Deployment | OWASP checklist verification |
| Post-Deployment | Monitor security events, alerting |

### Enforcement

- Security findings are **blocking** — deployment cannot proceed with unresolved High/Critical findings
- All security decisions logged in audit trail
- Security exceptions require documented justification and manager approval
