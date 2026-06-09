---
name: staged-workflow
description: Plan and coordinate multi-agent implementation work with contract-first sequencing, non-overlapping executor waves, review gates, and final verification. Use after planning and doc-grilling when a task needs multiple executors, shared protocol/API contracts, frontend/backend coordination, risk-sensitive fan-out, or explicit sequential-vs-parallel agent workflow.
---

# Staged Workflow

## Purpose

Use this after a technical plan is stable and any domain/documentation grilling has resolved major ambiguities.

This skill turns a plan into an execution workflow that multiple agents can run without stepping on each other.

## Core Rule

Do not fan out until shared contracts and conflict boundaries are explicit.

Contracts first. Foundations second. Parallel work third. Review before broad fan-out.

## Workflow

### 1. Confirm Inputs

Start from an existing plan, PRD, or issue set. If the plan is not technical enough, stop and ask for more planning before splitting execution.

Check for:

- shared protocol/API/type contracts
- database or migration boundaries
- frontend/backend ownership boundaries
- high-conflict files
- incident/security/regression constraints
- verification commands

### 2. Lock the Contract First

Create a Wave 0 contract task before implementation.

Define:

- command/event/API names
- required payload fields
- shared types or schema locations
- error/status semantics
- compatibility expectations
- files that own the contract

No executor should invent shared fields independently.

### 3. Split into Waves

Use this default sequence unless the project clearly needs a simpler version:

1. **Wave 0: Contract lock** — one owner defines shared protocol/types and updates the plan.
2. **Wave 1: Foundations** — sequential work for database/session/auth/core architecture changes.
3. **Wave 1.5: Review gate** — separate reviewer checks contract and foundation before fan-out.
4. **Wave 2: Parallel model/protocol work** — backend protocol, frontend state models, parsing, reducers.
5. **Wave 3: Parallel feature/UI work** — user-facing pieces that consume locked state/contracts.
6. **Wave 4: Integrated controls** — commands or behavior requiring both sides, such as cancel/steer/retry.
7. **Wave 5: Cross-cutting review and final verification** — incident review, security review, tests, manual path.

### 4. Add Non-Overlap Rules

For each executor task, specify:

- files it owns
- files it must not touch
- dependencies
- output/deliverable
- verification

Call out high-conflict files explicitly. If many tasks need the same large file, require an early helper/type extraction or serialize those tasks.

### 5. Add Review Gates

Insert a review gate before broad fan-out when any of these are true:

- shared API/protocol changed
- database/session boundaries changed
- security or incident-regression risk exists
- multiple executors depend on the same assumption
- one file is a central conflict point

The review agent must not be the same executor that made the changes under review.

### 6. Use Checkbox Task Format

Write tasks in this format:

```md
- [ ] **Area-N: Task name**
  - Files:
  - Depends on:
  - Must not touch:
  - Deliverable:
  - Verification:
```

For review gates:

```md
- [ ] **Review-N: Review name**
  - Reviewer: not the executor of the reviewed work.
  - Review:
  - Block fan-out until:
```

### 7. Final Verification

End with one cross-cutting verification task that includes:

- area tests
- build/typecheck commands
- integration or regression tests
- manual critical-path scenario
- incident/security review if relevant

## Output Shape

Update the plan with:

- `## Executor Work Items`
- `## Fan-Out Order and Non-Overlap Rules`
- wave checkboxes
- review gates
- final verification checklist

Keep it project-specific in the plan. Keep this skill generic.
