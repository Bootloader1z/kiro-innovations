# .kiro — AWS AI-DLC Rules (Extended)

This repository contains Kiro IDE steering rules and detailed rule files for the **AWS AI-DLC (AI-Driven Development Lifecycle)** framework, with custom extensions and an additional construction stage.

## What's Inside

```
.kiro/
├── steering/
│   ├── aws-aidlc-rules/
│   │   └── core-workflow.md          # Main AI-DLC workflow steering file
│   └── team-standards.md            # Tech stack & coding conventions (always loaded)
└── aws-aidlc-rule-details/
    ├── common/                       # Shared rules (validation, formatting, session continuity)
    ├── inception/                    # Inception phase stages
    ├── construction/                 # Construction phase stages (includes test-planning.md)
    ├── extensions/                   # Opt-in extension rules
    └── operations/                   # Operations phase (placeholder)
```

## Team Standards (Steering)

The file `.kiro/steering/team-standards.md` is automatically loaded into every Kiro session. It serves as the single source of truth for:

- **Tech stack** — Vue 3 + TypeScript + Tailwind + PrimeVue (frontend), Laravel 10 + PHP 8.2 (backend)
- **Architecture patterns** — folder structures for both frontend and backend
- **Coding conventions** — naming, component order, API call patterns, response formats
- **Formatting rules** — Prettier/ESLint config (frontend), Laravel Pint (backend)
- **Docker/Deployment** — container layout, ECR, MySQL + Redis

This ensures every team member gets consistent AI assistance that follows your established patterns.

---

## What We've Added to the AI-DLC Framework

### New Stage (Construction Phase)

| File | Type | What It Does |
|------|------|--------------|
| `test-planning.md` | Stage (mandatory) | Formal Test Plan document between NFR Design and Code Generation |

This changes the per-unit loop from:

**Before:**
```
Functional Design → NFR Req → NFR Design → Code Gen → Build & Test
```

**After:**
```
Functional Design → NFR Req → NFR Design → Test Planning → Code Gen → Build & Test
```

### New Extensions (Opt-In)

| Directory | Files | What It Enforces |
|-----------|-------|------------------|
| `extensions/qa/` | `qa-testing.md` + `.opt-in.md` | Automated quality gates, test pyramid, coverage targets, manual QA checklist, regression rules |
| `extensions/code-review/` | `code-review.md` + `.opt-in.md` | Reviewer requirements, 6-category checklist, feedback labels, PR standards, response SLAs |
| `extensions/deployment/` | `deployment-cicd.md` + `.opt-in.md` | GitHub Actions pipelines, environment strategy, rollback plan, release management, DORA metrics |
| `extensions/security/compliance/` | `security-compliance.md` + `.opt-in.md` | OWASP release checklist, scanning schedule, vulnerability SLAs, incident response |

## Usage

Clone this repo into your project's root as `.kiro/`:

```bash
git clone https://github.com/mblejano07/.kiro.git .kiro
```

Or add as a submodule:

```bash
git submodule add https://github.com/mblejano07/.kiro.git .kiro
```

Then open your project in Kiro IDE — the steering rules and rule details will be picked up automatically.

## Author

**mblejano07** — built with [Kiro IDE](https://kiro.dev) powered by Claude (Anthropic)

## License

MIT
