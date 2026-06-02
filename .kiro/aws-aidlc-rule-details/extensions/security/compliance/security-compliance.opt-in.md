# Security Compliance — Opt-In

**Extension**: Security Compliance Framework

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: Security Compliance Extension
Should the security compliance framework be enforced for this project? (This is separate from the Security Baseline rules — it adds release checklists, scanning schedules, vulnerability response SLAs, and incident response procedures.)

A) Yes — enforce all security compliance rules (OWASP checklist per release, scanning schedule, vulnerability SLAs, incident response)
B) Partial — enforce OWASP checklist and dependency scanning only (skip incident response and penetration testing requirements)
C) No — skip security compliance framework (Security Baseline rules still apply if enabled separately)
X) Other (please describe after [Answer]: tag below)

[Answer]: 
```
