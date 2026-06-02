# QA & Testing — Opt-In

**Extension**: Quality Assurance & Testing

## Opt-In Prompt

The following question is automatically included in the Requirements Analysis clarifying questions when this extension is loaded:

```markdown
## Question: QA & Testing Extension
Should QA and testing framework rules be enforced for this project?

A) Yes — enforce all QA rules as blocking constraints (automated gates, coverage targets, manual QA checklist, regression testing rules)
B) Partial — enforce automated gates only (lint, tests, build) but skip manual QA checklist requirements
C) No — skip all QA rules (suitable for quick prototypes or spikes)
X) Other (please describe after [Answer]: tag below)

[Answer]: 
```
