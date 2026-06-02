# Security Compliance Framework

## Overview

This extension adds release checklists, security scanning schedules, vulnerability response SLAs, and incident response procedures on top of the Security Baseline (SECURITY-01 through SECURITY-15) rules. It is a cross-cutting concern that applies across all AI-DLC phases.

**Enforcement**: At each applicable stage, the model MUST verify compliance with these rules before presenting the stage completion message to the user.

### Blocking Compliance Finding Behavior

A **blocking compliance finding** means:
1. The finding MUST be listed in the stage completion message under a "Compliance Findings" section with the COMPLIANCE rule ID and description
2. The stage MUST NOT present the "Continue to Next Stage" option until all blocking findings are resolved
3. The model MUST present only the "Request Changes" option with a clear explanation of what needs to change
4. The finding MUST be logged in `aidlc-docs/audit.md` with the COMPLIANCE rule ID, description, and stage context

If a COMPLIANCE rule is not applicable to the current project (e.g., COMPLIANCE-05 when no production deployment exists), mark it as **N/A** in the compliance summary — this is not a blocking finding.

### Default Enforcement

All rules in this document are **blocking** by default. If any rule's verification criteria are not met, it is a blocking compliance finding — follow the blocking finding behavior defined above.

### Partial Enforcement Mode

If Partial enforcement is selected (opt-in option B), only the OWASP release checklist and dependency scanning are enforced — i.e., **COMPLIANCE-02** and **COMPLIANCE-03** are blocking. Incident response (COMPLIANCE-05) and penetration testing requirements become advisory (non-blocking). Log the enforcement mode in `aidlc-docs/aidlc-state.md` under `## Extension Configuration`.

### Verification Criteria Format

Verification items in this document are plain bullet points describing compliance checks. They are distinct from the `- [ ]` / `- [x]` progress-tracking checkboxes used in stage plan files. Each item should be evaluated as compliant or non-compliant during review.

---

## Rule COMPLIANCE-01: Security Review Triggers

**Rule**: A dedicated security review is REQUIRED when changes touch any of the following areas:

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

**Verification**:
- Changes touching the above areas have a documented security review
- Security review findings are recorded before proceeding to the next stage
- High-risk changes are flagged during Requirements Analysis

---

## Rule COMPLIANCE-02: OWASP Top 10:2025 Compliance Checklist (Per Release)

**Rule**: Every release MUST be verified against the following OWASP Top 10:2025 checklist. All items must pass before deployment.

### A01:2025 — Broken Access Control

- [ ] All routes require authentication (except explicit public routes) (see SECURITY-08)
- [ ] RBAC enforced server-side on every endpoint (see SECURITY-08)
- [ ] Object-level authorization (IDOR prevention — users access only their own resources) (see SECURITY-08)
- [ ] Admin functions have explicit role checks (see SECURITY-08)
- [ ] CORS restricted to allowed origins (no wildcard on authenticated endpoints) (see SECURITY-08)
- [ ] Rate limiting on public-facing endpoints (see SECURITY-11)

### A02:2025 — Security Misconfiguration

- [ ] Content-Security-Policy: `default-src 'self'` (minimum) (see SECURITY-04)
- [ ] Strict-Transport-Security: `max-age=31536000; includeSubDomains` (see SECURITY-04)
- [ ] X-Content-Type-Options: `nosniff` (see SECURITY-04)
- [ ] X-Frame-Options: `DENY` (see SECURITY-04)
- [ ] Referrer-Policy: `strict-origin-when-cross-origin` (see SECURITY-04)

### A03:2025 — Software Supply Chain Failures

- [ ] Lock files committed (composer.lock, package-lock.json) (see SECURITY-10)
- [ ] No unused dependencies in project (see SECURITY-10)
- [ ] `composer audit` passes (zero critical/high)
- [ ] `npm audit` passes (zero critical/high)
- [ ] Docker images use pinned versions (no `:latest` in production) (see SECURITY-10)
- [ ] Dependencies sourced from official registries only (see SECURITY-10)

### A04:2025 — Cryptographic Failures

- [ ] Sensitive data encrypted at rest (database encryption enabled) (see SECURITY-01)
- [ ] All data in transit over TLS 1.2+ (see SECURITY-01)
- [ ] PII minimized (only collect what's needed)
- [ ] Data retention policies defined and enforced
- [ ] Audit trail for critical data changes (who, what, when) (see SECURITY-13)
- [ ] Backups encrypted and access-controlled

### A05:2025 — Injection

- [ ] All inputs validated server-side (Form Requests / validation schemas) (see SECURITY-05)
- [ ] String inputs have max-length constraints (see SECURITY-05)
- [ ] File uploads validated (MIME type allowlist, size limits, no executables)
- [ ] No raw user input in SQL queries (parameterized queries only) (see SECURITY-05)
- [ ] No raw user input in shell commands or file paths
- [ ] Request body size limits configured (see SECURITY-05)
- [ ] HTML/script content escaped or rejected (XSS prevention) (see SECURITY-05)

### A06:2025 — Insecure Design

- [ ] Security logic isolated in dedicated modules (not scattered) (see SECURITY-11)
- [ ] Defense in depth (multiple layers: validation + auth + encryption) (see SECURITY-11)
- [ ] Fail closed (errors deny access, never fail open) (see SECURITY-15)
- [ ] Least privilege (roles have minimum necessary permissions) (see SECURITY-06)
- [ ] Misuse cases considered in design (not just happy path) (see SECURITY-11)

### A07:2025 — Authentication Failures

- [ ] Passwords hashed with adaptive algorithm (bcrypt cost 12+) (see SECURITY-12)
- [ ] Account lockout after failed attempts (5 attempts, 15-min lockout) (see SECURITY-12)
- [ ] Session timeout configured (2h idle, 8h absolute) (see SECURITY-12)
- [ ] Session invalidated on logout (see SECURITY-12)
- [ ] Cookies: Secure, HttpOnly, SameSite=Lax (see SECURITY-12)
- [ ] No credentials in source code or logs (see SECURITY-12)
- [ ] MFA supported for admin accounts (when applicable) (see SECURITY-12)

### A08:2025 — Software or Data Integrity Failures

- [ ] No unsafe deserialization of untrusted input (see SECURITY-13)
- [ ] External scripts include SRI integrity attributes when loaded from CDNs (see SECURITY-13)
- [ ] CI/CD pipeline definitions are access-controlled and changes are auditable (see SECURITY-13)
- [ ] Critical data changes are logged with actor, timestamp, and before/after values (see SECURITY-13)

### A09:2025 — Security Logging & Alerting Failures

- [ ] Production errors return generic messages only (see SECURITY-09)
- [ ] No stack traces, SQL queries, or file paths exposed to users (see SECURITY-09)
- [ ] Global exception handler configured (see SECURITY-15)
- [ ] Security events logged: failed logins, access denied, privilege changes (see SECURITY-14)
- [ ] Logs include: timestamp, request_id, user_id, action (see SECURITY-03)
- [ ] Logs do NOT contain: passwords, tokens, session IDs, PII (see SECURITY-03)
- [ ] Log retention: minimum 90 days (see SECURITY-14)

### A10:2025 — Mishandling of Exceptional Conditions

- [ ] All external calls have explicit error handling (see SECURITY-15)
- [ ] System fails closed on error (see SECURITY-15)
- [ ] Resources cleaned up in error paths (see SECURITY-15)
- [ ] Global error handler configured at application entry point (see SECURITY-15)
- [ ] No unhandled promise rejections or uncaught exceptions in production (see SECURITY-15)

**Verification**:
- All checklist items are marked as passing before release
- Non-passing items are documented with justification or remediation plan
- Checklist completion is recorded in the stage completion summary

---

## Rule COMPLIANCE-03: Security Scanning Schedule

**Rule**: The following security scans MUST be performed on the defined schedule:

| Scan Type | Frequency | Tool | Owner |
|-----------|-----------|------|-------|
| Dependency vulnerabilities | Every PR + weekly | `composer audit`, `npm audit` | CI Pipeline |
| Static analysis (security) | Every PR | GrumPHP / Enlightn Security Checker | CI Pipeline |
| OWASP Top 10 review | Per release | Manual checklist (COMPLIANCE-02) | Security reviewer |
| Dynamic application testing | Monthly | OWASP ZAP | Security team |
| Penetration testing | Quarterly | External firm or OWASP ZAP | Security team |
| Infrastructure review | Per deployment change | Manual review | DevOps |
| Secret scanning | Every PR | GitLab CI secret scanning / gitleaks | CI Pipeline |

**Verification**:
- CI pipeline includes dependency audit and secret scanning steps
- Scan results are reviewed and findings triaged within defined SLAs
- Quarterly penetration testing is scheduled and results documented

---

## Rule COMPLIANCE-04: Vulnerability Response SLA

**Rule**: Discovered vulnerabilities MUST be handled within the following SLAs:

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

**Verification**:
- All discovered vulnerabilities are tracked with severity classification
- Patch timelines comply with the SLA table above
- Root cause analysis is documented for Critical and High findings

---

## Rule COMPLIANCE-05: Security Incident Response

**Rule**: Security incidents MUST be classified and handled according to the following framework:

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

**Verification**:
- Incident classification is applied to all security events
- Response actions are executed within the defined timelines
- Post-incident reviews are conducted for P1 and P2 incidents

---

## Enforcement Integration

This extension complements — does not replace — the Security Baseline (SECURITY-01 through SECURITY-15) rules. The baseline rules enforce secure coding and architecture constraints at every stage; this extension adds process-level controls (checklists, scanning cadence, SLAs, and incident response).

| Stage | Applicable Rules | Enforcement |
|-------|-----------------|-------------|
| Requirements Analysis | COMPLIANCE-01 | Identify security-sensitive changes and data sensitivity; trigger security review |
| User Stories | COMPLIANCE-01 | Include misuse cases and abuse scenarios |
| Functional Design | COMPLIANCE-01 | Threat modeling (identify data flows and trust boundaries) |
| NFR Requirements | COMPLIANCE-01, COMPLIANCE-02 | Define security controls and compliance targets |
| NFR Design | COMPLIANCE-01, COMPLIANCE-02 | Design security patterns aligned with OWASP checklist |
| Code Generation | COMPLIANCE-01 | Apply secure coding patterns; generate security tests |
| Build and Test | COMPLIANCE-02, COMPLIANCE-03 | OWASP checklist verification + security scans |
| Operations | COMPLIANCE-04, COMPLIANCE-05 | Vulnerability SLA enforcement + incident response |

At each stage:
- Evaluate all applicable COMPLIANCE rule verification criteria against the artifacts produced
- Include a "Security Compliance" section in the stage completion summary listing each rule as compliant, non-compliant, or N/A
- If any rule is non-compliant, this is a blocking compliance finding — follow the blocking finding behavior defined in the Overview
- Check the Enabled status in `aidlc-docs/aidlc-state.md` under `## Extension Configuration` before enforcing; if disabled, skip enforcement and log the skip reason
