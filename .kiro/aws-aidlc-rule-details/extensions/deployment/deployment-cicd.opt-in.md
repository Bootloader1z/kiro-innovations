# Deployment & CI/CD — Opt-In

**Extension**: Deployment & CI/CD (CI-provider-agnostic; the team's actual CI is defined in `team-standards.md`)

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: Deployment & CI/CD Extension
Should deployment and CI/CD framework rules be enforced for this project?

A) Yes — enforce all deployment rules (CI pipeline, environment strategy, rollback plan, monitoring, DORA metrics)
B) Partial — enforce CI pipeline only (lint, test, build gates) but skip deployment automation and monitoring
C) No — skip deployment rules (suitable for local-only development or projects with existing CI/CD)
X) Other (please describe after [Answer]: tag below)

[Answer]: 
```
