# Test Planning Framework

**Purpose**: Defines the Test Planning stage as a separate gate in the AI-DLC CONSTRUCTION phase, producing a formal Test Plan document before code generation begins.  
**Position in Workflow**: After NFR Design, before Code Generation (per-unit).

---

## 1. Stage Position in AI-DLC

```
CONSTRUCTION PHASE (per-unit loop):

  Functional Design        ← WHAT the unit does (business logic, rules)
  NFR Requirements         ← WHAT quality targets to meet
  NFR Design               ← HOW to meet quality targets (patterns)
  ┌────────────────────┐
  │  TEST PLANNING     │   ← WHAT to test, HOW to test, PASS/FAIL criteria
  └────────────────────┘
  Code Generation          ← Implements code + tests per the Test Plan
  Build and Test           ← Executes tests, reports results against plan
```

---

## 2. When to Execute

**ALWAYS execute** for each unit. The depth adapts:

| Unit Complexity | Test Plan Depth |
|----------------|-----------------|
| Simple (CRUD, few rules) | Lightweight — test scenarios + coverage targets |
| Medium (business logic, workflows) | Standard — full test plan with scenarios, data, PBT properties |
| Complex (state machines, integrations, security-critical) | Comprehensive — detailed plan with edge cases, load scenarios, security tests |

---

## 3. Test Plan Document Structure

Create `aidlc-docs/construction/{unit-name}/test-plan/test-plan.md`:

### 3.1 Test Scope

```markdown
## Test Scope

### In Scope
- [List features/components being tested in this unit]
- [List business rules to verify]
- [List integrations to validate]

### Out of Scope
- [Features tested in other units]
- [External systems not mocked]
- [Performance testing (if deferred)]
```

### 3.2 Test Strategy

```markdown
## Test Strategy

### Test Types Required

| Type | Coverage Target | Framework | Rationale |
|------|----------------|-----------|-----------|
| Unit Tests | 80%+ business logic | Pest (PHP) | Verify individual services/methods |
| Integration Tests | Key cross-domain flows | Pest (Feature) | Verify components work together |
| Property-Based Tests | All identified properties | Eris / fast-check | Verify invariants hold universally |
| Frontend Tests | Key interactions | Vitest | Verify UI behavior |
| E2E Tests | Critical paths | Laravel Dusk (optional) | Verify full user journeys |
```

### 3.3 Test Scenarios (Derived from User Stories)

```markdown
## Test Scenarios

### Scenario Group: [Feature/Story Name]

| ID | Scenario | Input | Expected Result | Priority | Type |
|----|----------|-------|-----------------|----------|------|
| TS-01 | [Happy path description] | [Input data] | [Expected outcome] | High | Unit |
| TS-02 | [Error case description] | [Invalid input] | [Error response] | High | Unit |
| TS-03 | [Edge case description] | [Boundary data] | [Expected behavior] | Medium | Unit |
| TS-04 | [Integration scenario] | [Cross-domain input] | [End-to-end result] | High | Integration |
```

### 3.4 Property-Based Test Specifications

```markdown
## Property-Based Tests

### Properties Identified (from Functional Design)

| ID | Property | Category | Generator | Assertion |
|----|----------|----------|-----------|-----------|
| PBT-01 | [Property name] | Round-trip | [Input generator description] | f_inv(f(x)) == x |
| PBT-02 | [Property name] | Invariant | [Input generator description] | measure(f(x)) == measure(x) |
| PBT-03 | [Property name] | Idempotence | [Input generator description] | f(f(x)) == f(x) |
| PBT-04 | [Property name] | Stateful | [Command generator description] | model matches system |
```

### 3.5 Test Data Requirements

```markdown
## Test Data

### Fixtures / Seeders Required
- [List database seeders needed for tests]
- [List factory definitions needed]

### Test Data Constraints
- [Data that must exist before tests run]
- [Data isolation requirements (RefreshDatabase, transactions)]

### Generators (for PBT)
- [Domain-specific generators to create]
- [Constraints generators must respect]
```

### 3.6 Pass/Fail Criteria

```markdown
## Pass/Fail Criteria

### Unit MUST NOT proceed to Code Generation if:
- Test scenarios are incomplete (not all acceptance criteria covered)
- PBT properties are not identified for applicable code
- Coverage targets are not defined

### Build and Test stage PASSES if:
- All unit tests pass (100% pass rate)
- All integration tests pass
- All PBT tests pass (with seed logged)
- Coverage meets minimum threshold (80% business logic)
- No security test failures
- Performance within NFR targets (if tested)

### Build and Test stage FAILS if:
- Any test fails
- Coverage below threshold
- Security scan finds critical/high vulnerability
- PBT discovers a failing property (must be fixed, not suppressed)
```

---

## 4. Inputs to Test Planning

The Test Plan is derived from these prior artifacts:

| Source | What It Provides |
|--------|-----------------|
| User Stories (acceptance criteria) | Functional test scenarios (Given/When/Then) |
| Business Rules | Validation and constraint test cases |
| Functional Design (PBT properties) | Property-based test specifications |
| NFR Requirements | Performance targets, security requirements |
| NFR Design (patterns) | Integration test scenarios, security test cases |
| Domain Entities | Test data requirements, factory definitions |

---

## 5. Outputs of Test Planning

| Artifact | Location | Consumed By |
|----------|----------|-------------|
| Test Plan document | `aidlc-docs/construction/{unit-name}/test-plan/test-plan.md` | Code Generation |
| Test scenario matrix | Embedded in test plan | Code Generation (what tests to write) |
| PBT specifications | Embedded in test plan | Code Generation (property tests) |
| Test data requirements | Embedded in test plan | Code Generation (factories/seeders) |
| Pass/fail criteria | Embedded in test plan | Build and Test (verification) |

---

## 6. Execution Steps

### Step 1: Load Context
- Read Functional Design artifacts (business rules, PBT properties)
- Read NFR Requirements (performance targets, security controls)
- Read NFR Design (patterns, logical components)
- Read User Stories assigned to this unit (acceptance criteria)

### Step 2: Generate Test Scenarios
- Convert each acceptance criterion into one or more test scenarios
- Identify happy path, error path, and edge case scenarios
- Map scenarios to test types (unit, integration, PBT, E2E)
- Assign priority (High/Medium/Low)

### Step 3: Specify PBT Properties
- List all properties identified in Functional Design
- Define generator requirements for each property
- Specify assertion format
- Note any stateful testing needs

### Step 4: Define Test Data
- List required seeders and factories
- Define generator constraints for PBT
- Specify data isolation strategy

### Step 5: Set Pass/Fail Criteria
- Define coverage thresholds
- Define performance benchmarks (if applicable)
- Define security scan requirements

### Step 6: Present for Approval

```markdown
# 📋 Test Plan Complete - [unit-name]

[Summary of test plan contents]

> **📋 REVIEW REQUIRED:**
> Please examine the test plan at: `aidlc-docs/construction/[unit-name]/test-plan/test-plan.md`

> **🚀 WHAT'S NEXT?**
>
> **You may:**
>
> 🔧 **Request Changes** - Modify test scenarios or coverage targets
> ✅ **Continue to Next Stage** - Approve test plan and proceed to **Code Generation**
```

### Step 7: Wait for Approval
- Do not proceed to Code Generation until test plan is approved
- If changes requested, update and re-present

---

## 7. Relationship to Other Frameworks

| Framework | Relationship to Test Planning |
|-----------|------------------------------|
| `qa-testing.md` | Defines quality gates that the test plan must satisfy |
| `security-compliance.md` | Defines security tests that must be included in the plan |
| `code-review.md` | Reviewers verify tests match the approved test plan |
| `deployment-cicd.md` | CI pipeline executes the tests defined in the plan |

---

## 8. Example Test Plan (Abbreviated)

```markdown
# Test Plan — Unit 2: Ticket Management

## Test Scope
- Ticket CRUD operations
- Status state machine transitions
- Auto-assignment logic
- Comment system (public + internal)
- Search and filtering

## Test Strategy
| Type | Target | Framework |
|------|--------|-----------|
| Unit | 80% services | Pest |
| Integration | 5 cross-domain flows | Pest Feature |
| PBT | 4 properties | Eris |

## Key Scenarios
| ID | Scenario | Priority |
|----|----------|----------|
| TS-01 | Employee creates incident ticket → reference generated, status=open | High |
| TS-02 | Invalid status transition rejected (open → pending) | High |
| TS-03 | Employee cannot see internal comments | High |
| TS-04 | Agent queue sorted by priority | Medium |

## PBT Properties
| Property | Category | Assertion |
|----------|----------|-----------|
| Status machine validity | Stateful | Random transitions always reach valid state |
| Reference uniqueness | Invariant | No two tickets share reference |

## Pass/Fail
- All 15+ test scenarios pass
- Coverage ≥ 80% on TicketService
- PBT: 100 iterations minimum, seed logged
```
