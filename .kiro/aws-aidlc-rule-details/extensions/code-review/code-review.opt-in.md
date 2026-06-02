# Code Review — Opt-In

**Extension**: Code Review Framework

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: Code Review Extension
Should code review framework rules be enforced for this project?

A) Yes — enforce all code review rules (reviewer requirements, checklist, SLA, PR standards)
B) Partial — enforce checklist and PR standards only (skip reviewer count and SLA requirements for solo developers)
C) No — skip code review rules (suitable for solo projects or learning exercises)
X) Other (please describe after [Answer]: tag below)

[Answer]: 
```
