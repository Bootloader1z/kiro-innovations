# kiro-innovations

Kiro IDE workspace with AI-DLC framework, Graphify knowledge graph, and comprehensive team standards for full-stack development.

## What's Inside

```
.kiro/
├── settings/
│   └── mcp.json                      # MCP servers (Playwright)
├── skills/
│   └── graphify/SKILL.md             # Graphify knowledge graph skill
├── steering/
│   ├── graphify.md                   # Always-on graph query steering
│   ├── subagent-delegation.md        # Parallel work delegation rules
│   ├── team-standards.md             # Full tech stack & API design (always loaded)
│   └── aws-aidlc-rules/
│       └── core-workflow.md          # AI-DLC adaptive workflow engine
└── aws-aidlc-rule-details/
    ├── common/                       # Shared rules (validation, formatting, session continuity)
    ├── inception/                    # Inception phase stages
    ├── construction/                 # Construction phase stages (includes test-planning)
    ├── extensions/                   # Opt-in extension rules (security, QA, deployment, etc.)
    └── operations/                   # Operations phase (placeholder)
```

## Features

### AI-DLC (AI-Driven Development Lifecycle)

A structured, adaptive software development workflow that guides AI assistants through:

- **Inception Phase** — workspace detection, reverse engineering, requirements, user stories, workflow planning, application design
- **Construction Phase** — functional design, NFR assessment, test planning, code generation, build & test
- **Operations Phase** — placeholder for deployment and monitoring (future)

### Graphify Integration

Knowledge graph skill that maps your codebase into a queryable graph. Use `graphify query "..."` for architecture questions instead of grepping files.

### Team Standards

Comprehensive single-source-of-truth document covering:

- **Frontend**: Vue 3 + TypeScript + Tailwind CSS + PrimeVue (webkit-spa-prime)
- **Backend**: Laravel 10 + PHP 8.2 + Sanctum + JWT (webkit-api)
- **Full API route map** with auth, users, profile, notifications, dashboard, audits, roles
- **Security pipeline** — rate limiting, CORS, Sanctum, MFA gate, role/permission, policies
- **Audit system** — polymorphic audit trail with queued logging
- **Database schema** — users, roles, permissions, audits, notifications, tokens
- **Core service functions** — AuthService, MfaService, UserService, NotificationService, DashboardService, AuditService
- **Enums** — Role, Permission, AuditAction, ApiErrorCode
- **Infrastructure** — Docker Compose, Redis queues, GitLab CI, AWS ECR/ECS

### Subagent Delegation

Rules for parallelizing work across independent implementation units using context-gatherer and general-task-execution subagents.

### MCP Servers

- **Playwright** — browser automation for testing and interaction

## AI-DLC Extensions (Opt-In)

| Extension | Rules | Enforces |
|-----------|-------|----------|
| QA Testing | `QA-01..07` | Test pyramid, coverage targets, quality gates |
| Code Review | `CR-01..06` | PR standards, checklist, response SLAs |
| Deployment | `DEPLOY-01..07` | CI pipeline gates, rollback plans, DORA metrics |
| Security Baseline | `SECURITY-01..15` | OWASP Top 10:2025 mapped controls |
| Security Compliance | `COMPLIANCE-01..05` | Scanning schedule, vulnerability SLAs |
| Property-Based Testing | `PBT-01..10` | Invariants, generators, shrinking |

## Setup

### Clone into your project

```bash
git clone https://github.com/Bootloader1z/kiro-innovations.git .kiro
```

### Or add as submodule

```bash
git submodule add https://github.com/Bootloader1z/kiro-innovations.git .kiro
```

### Prerequisites

- [Kiro IDE](https://kiro.dev)
- Python 3.10+ with `uv` (for Graphify)
- Node.js 18+ (for Playwright MCP)

### Install Graphify

```bash
uv tool install graphifyy
graphify kiro install
```

## Usage

Open your project in Kiro IDE — steering rules load automatically. Then:

- Type `/graphify .` to build a knowledge graph of your codebase
- Start a development request to trigger the AI-DLC workflow
- Use `graphify query "..."` for architecture questions

## Versioning

| Branch | Description |
|--------|-------------|
| `main` | Stable release |
| `v1.0.0/initial-setup` | Initial AI-DLC + Graphify + team standards |

## Author

**Bootloader1z** — built with [Kiro IDE](https://kiro.dev)

## License

MIT
