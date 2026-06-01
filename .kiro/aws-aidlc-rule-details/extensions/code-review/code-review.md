# Code Review Framework

**Purpose**: Defines code review standards, processes, and expectations for the AI-DLC workflow.  
**Applies to**: All code changes before merge to main branch.

---

## 1. Review Requirements

### When is Review Required?

ALL code changes require review before merge. No exceptions.

### Reviewer Count by Change Type

| Change Type | Minimum Reviewers | Review Depth |
|-------------|-------------------|--------------|
| Bug fix (isolated, < 50 lines) | 1 reviewer | Focused review |
| Feature (single domain) | 1 reviewer | Full review |
| Feature (cross-domain) | 2 reviewers | Full review |
| Security-sensitive change | 2 reviewers (1 security-aware) | Deep review |
| Database migration | 2 reviewers | Full review + rollback plan |
| Infrastructure/deployment | 2 reviewers | Full review |
| Hotfix (production emergency) | 1 reviewer (post-deploy review by 2nd) | Expedited |

---

## 2. Code Review Checklist

Every reviewer MUST evaluate against these categories:

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

---

## 3. Review Feedback Categories

Use these labels when providing feedback:

| Label | Meaning | Blocking? | Example |
|-------|---------|-----------|---------|
| **MUST FIX** | Security issue, bug, or standards violation | Yes | "SQL injection vulnerability" |
| **SHOULD FIX** | Significant quality improvement | Yes (unless justified) | "Missing error handling on API call" |
| **CONSIDER** | Suggestion for improvement | No | "Could extract this into a helper" |
| **NIT** | Style preference, minor formatting | No | "Prefer const over let here" |
| **QUESTION** | Seeking clarification | No | "Why was this approach chosen over X?" |
| **PRAISE** | Highlighting good work | No | "Nice use of the strategy pattern here" |

---

## 4. Review Response Time SLA

| Priority | First Review Within | Resolution Within |
|----------|--------------------|--------------------|
| Critical (production fix) | 2 hours | 4 hours |
| High (blocking other work) | 4 hours | 1 business day |
| Normal | 1 business day | 2 business days |
| Low (refactoring, docs) | 2 business days | 5 business days |

---

## 5. PR Submission Standards

### Before Submitting a PR

The author MUST:
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

---

## 6. Review Best Practices

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

---

## 7. AI-DLC Integration Points

| AI-DLC Phase | Code Review Activity |
|-------------|---------------------|
| Code Generation | Self-review generated code against checklist |
| Build and Test | Verify all gates pass before review request |
| Code Review | Full review per this framework |
| Pre-Deployment | Final approval gate before merge |

### Enforcement

- No code merges to main without at least 1 approval
- MUST FIX findings block merge until resolved
- Review history preserved in PR for audit trail
