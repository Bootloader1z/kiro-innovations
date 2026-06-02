# Code Review Framework

## Overview

This document defines mandatory code review standards, processes, and expectations for the AI-DLC workflow. These rules are cross-cutting constraints that apply across applicable AI-DLC stages — they are not optional guidance but hard constraints that stages MUST enforce when generating code, verifying build readiness, and gating merge operations.

All code changes require review before merge to the main branch. No exceptions.

**Enforcement**: At each applicable stage, the model MUST verify compliance with these rules before presenting the stage completion message to the user.

### Blocking Code Review Finding Behavior

A **blocking code review finding** means:
1. The finding MUST be listed in the stage completion message under a "Code Review Findings" section with the CR rule ID and description
2. The stage MUST NOT present the "Continue to Next Stage" option until all blocking findings are resolved
3. The model MUST present only the "Request Changes" option with a clear explanation of what needs to change
4. The finding MUST be logged in `aidlc-docs/audit.md` with the CR rule ID, description, and stage context

If a CR rule is not applicable to the current change, mark it as **N/A** in the compliance summary — this is not a blocking finding.

### Default Enforcement

All rules in this document are **blocking** by default. If any rule's verification criteria are not met, it is a blocking code review finding — follow the blocking finding behavior defined above.

### Partial Enforcement Mode

If the user selected **Partial** during opt-in (option B), only the checklist and PR-submission rules are enforced as blocking — i.e. CR-02 and CR-04. Reviewer-count (CR-01) and SLA (CR-05) rules become advisory (non-blocking). CR-03 and CR-06 remain informational guidance. Log the enforcement mode in `aidlc-docs/aidlc-state.md` under `## Extension Configuration`.

### Verification Criteria Format

Verification items in this document are plain bullet points describing compliance checks. Each item should be evaluated as compliant or non-compliant during review.

---

## Rule CR-01: Review Requirements

**Rule**: ALL code changes require review before merge. The minimum number of reviewers depends on the change type:

| Change Type | Minimum Reviewers | Review Depth |
|-------------|-------------------|--------------|
| Bug fix (isolated, < 50 lines) | 1 reviewer | Focused review |
| Feature (single domain) | 1 reviewer | Full review |
| Feature (cross-domain) | 2 reviewers | Full review |
| Security-sensitive change | 2 reviewers (1 security-aware) | Deep review |
| Database migration | 2 reviewers | Full review + rollback plan |
| Infrastructure/deployment | 2 reviewers | Full review |
| Hotfix (production emergency) | 1 reviewer (post-deploy review by 2nd) | Expedited |

**Verification**:
- The change has been assigned the minimum number of reviewers for its change type
- All assigned reviewers have completed their review before merge
- Security-sensitive changes have at least one security-aware reviewer
- Database migrations include a documented rollback plan

---

## Rule CR-02: Code Review Checklist

**Rule**: Every reviewer MUST evaluate the change against all six categories below. Authors MUST self-review against this checklist before submitting.

### Functional Correctness
- [ ] Does the code solve the stated requirement?
- [ ] Are edge cases handled (null, empty, boundary values)?
- [ ] Are error paths covered with appropriate responses?
- [ ] Does it match the acceptance criteria from the user story?
- [ ] No unintended side effects on existing functionality?

### Architecture & Design
- [ ] Follows domain boundaries (no cross-domain leakage)?
- [ ] Business logic is in services (not controllers)?
- [ ] Dependencies are injected (testable)?
- [ ] No unnecessary coupling between components?
- [ ] Follows established patterns in the codebase?
- [ ] No over-engineering (YAGNI — You Aren't Gonna Need It)?

### Security
- [ ] All inputs validated (Form Requests)?
- [ ] Authorization checked (Policies/Middleware)?
- [ ] No secrets or credentials in code?
- [ ] No raw SQL with user input (parameterized queries only)?
- [ ] Error messages don't leak internal details?
- [ ] RBAC enforced server-side (not just UI hiding)?

### Testing
- [ ] New behavior has tests?
- [ ] Tests are meaningful (not just asserting true)?
- [ ] Edge cases covered in tests?
- [ ] PBT properties identified for algorithmic/stateful code?
- [ ] No test pollution (tests are independent)?
- [ ] Tests pass locally before PR submission?

### Maintainability
- [ ] Clear, descriptive naming (variables, functions, classes)?
- [ ] No code duplication (DRY — extract shared logic)?
- [ ] Appropriate comments (explain WHY, not WHAT)?
- [ ] Single responsibility (methods < 30 lines, classes < 300 lines)?
- [ ] Type hints on all parameters and return types?
- [ ] Dead code removed (no commented-out blocks)?

### Performance
- [ ] No N+1 queries (eager loading used)?
- [ ] Database queries use appropriate indexes?
- [ ] No unnecessary data loaded (select specific columns)?
- [ ] Pagination used for list endpoints?
- [ ] Caching considered for expensive operations?
- [ ] No blocking operations in request cycle?

**Verification**:
- All six checklist categories have been evaluated by the reviewer
- Any unchecked item has a documented justification or is marked N/A
- The author performed a self-review before submitting the PR

---

## Rule CR-03: Review Feedback Categories

**Rule**: Reviewers MUST use the following labels when providing feedback. Only MUST FIX and SHOULD FIX are blocking:

| Label | Meaning | Blocking? | Example |
|-------|---------|-----------|---------|
| **MUST FIX** | Security issue, bug, or standards violation | Yes | "SQL injection vulnerability" |
| **SHOULD FIX** | Significant quality improvement | Yes (unless justified) | "Missing error handling on API call" |
| **CONSIDER** | Suggestion for improvement | No | "Could extract this into a helper" |
| **NIT** | Style preference, minor formatting | No | "Prefer const over let here" |
| **QUESTION** | Seeking clarification | No | "Why was this approach chosen over X?" |
| **PRAISE** | Highlighting good work | No | "Nice use of the strategy pattern here" |

**Verification**:
- All review comments use one of the defined feedback labels
- MUST FIX findings are resolved before merge
- SHOULD FIX findings are resolved or have documented justification before merge
- Non-blocking feedback (CONSIDER, NIT, QUESTION, PRAISE) does not block merge

---

## Rule CR-04: PR Submission Standards

**Rule**: Before submitting a PR, the author MUST:
- [ ] Self-review the diff (catch obvious issues)
- [ ] Ensure all CI gates pass (lint, tests, build)
- [ ] Write a clear PR description (what, why, how)
- [ ] Link to the related user story or issue
- [ ] Keep PR size manageable (< 400 lines preferred, < 800 max)
- [ ] Split large features into incremental PRs

### PR Description Template

```markdown
## What
[Brief description of the change]

## Why
[Link to user story/issue, business context]

## How
[Technical approach, key decisions made]

## Testing
[How was this tested? What scenarios were covered?]

## Screenshots (if UI change)
[Mobile + Desktop screenshots]

## Checklist
- [ ] Tests added/updated
- [ ] Documentation updated (if needed)
- [ ] No breaking changes (or migration path documented)
- [ ] Self-reviewed the diff
```

**Verification**:
- PR description follows the template structure (What, Why, How, Testing sections present)
- All CI gates pass before review is requested
- PR size is within limits (< 800 lines; > 400 lines has justification)
- Related user story or issue is linked
- Author has self-reviewed the diff (confirmed in checklist)

---

## Rule CR-05: Review Response Time SLA

**Rule**: Reviewers MUST respond within the SLA defined by the priority of the change:

| Priority | First Review Within | Resolution Within |
|----------|--------------------|--------------------|
| Critical (production fix) | 2 hours | 4 hours |
| High (blocking other work) | 4 hours | 1 business day |
| Normal | 1 business day | 2 business days |
| Low (refactoring, docs) | 2 business days | 5 business days |

**Verification**:
- The PR has an assigned priority level
- First review was provided within the SLA for the assigned priority
- All blocking findings were resolved within the resolution SLA
- Stale PRs (exceeding SLA) are escalated or reassigned

---

## Rule CR-06: Review Best Practices & Anti-Patterns

**Rule**: Reviewers and authors MUST follow these practices and avoid the listed anti-patterns.

### For Reviewers
- Review within SLA — don't block teammates
- Be specific — point to the exact line and suggest a fix
- Explain WHY something should change, not just WHAT
- Distinguish blocking vs non-blocking feedback clearly
- Approve with minor nits rather than blocking on style
- If unsure, ask a question rather than assuming

### For Authors
- Keep PRs small and focused (one concern per PR)
- Respond to all feedback (even if just "acknowledged")
- Don't take feedback personally — it's about the code
- If you disagree, explain your reasoning respectfully
- Resolve conversations after addressing feedback

### Review Anti-Patterns (Avoid)
- ❌ Rubber-stamping (approving without reading)
- ❌ Bikeshedding (debating trivial style choices)
- ❌ Gatekeeping (blocking on personal preferences)
- ❌ Drive-by reviews (commenting without context)
- ❌ Stale PRs (letting reviews sit for days)

**Verification**:
- Review comments demonstrate engagement with the code (not rubber-stamping)
- Feedback is specific and actionable (not vague or style-only blocking)
- Author has responded to all review comments
- No anti-pattern behaviors are evident in the review thread

---

## Enforcement Integration

| Stage | Applicable Rules | Enforcement |
|-------|-----------------|-------------|
| Code Generation | CR-02 | Self-review generated code against checklist |
| Build and Test | CR-04 | CI gates pass before review is requested |
| Operations | CR-01, CR-03, CR-05, CR-06 | Peer review + merge gate enforcement |

At each stage:
- Evaluate all applicable CR rule verification criteria against the artifacts produced
- Include a "Code Review Compliance" section in the stage completion summary listing each applicable rule as compliant, non-compliant, or N/A
- If any rule is non-compliant, this is a blocking code review finding — follow the blocking finding behavior defined in the Overview
- Check the extension's `Enabled` status in `aidlc-docs/aidlc-state.md` under `## Extension Configuration` before enforcing; if disabled, skip enforcement and log the skip in `aidlc-docs/audit.md`
