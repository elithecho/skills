---
name: plan
description: Creates self-correcting execution plans for architecture-level or risky work before implementation. Use when a task is cross-component, risky, unclear, mentions planning gate, critique the plan, adversarial plan review, architect before coding, or asks how to plan before coding.
---

# Plan

## Purpose

Use this skill when implementation should not start until the work has a concrete plan that has survived adversarial critique.

Planning-gate work usually has:

- Architecture or boundary changes
- Many affected components
- Unclear ownership or sequencing

Do not use this skill as a substitute for coding. It produces a plan, resolves critique, and then hands execution to an implementation workflow such as `orchestrate`.

## Workflow

1. Explore before planning.
   - Map the relevant codepath, data flow, tests, owners, dependencies, and constraints.
   - Do not assign implementation until exploration is summarized.

2. Ask `architect` for the first plan.
   - Include the user goal, explored context, likely edit surface, constraints, and verification needs.
   - Require sequencing, ownership boundaries, rollback or migration concerns, and risk flags.
   - Require a clear definition of what proves completion.

3. Ask `critic` to challenge the plan.
   - Focus on missed requirements, hidden assumptions, sequencing risks, unsafe rollout, weak verification, and overbroad scope.
   - Treat blocking critique as required work, not advisory commentary.

4. Self-correct the plan.
   - Send critique back through the planning lane to update the plan.
   - Repeat architect update then critic review until there are no blocking critiques.
   - If critique reveals the task is smaller than expected, switch to `orchestrate` or local execution.
   - If critique reveals missing requirements, ask the user only for decisions that cannot be safely inferred.

5. Hand off implementation.
   - Send the final plan and critique resolution to the executor or `orchestrate` workflow.
   - Do not skip post-implementation review and verification.

## Architect Prompt Shape

```text
Create an execution plan for this architecture-level or risky task.

Goal:
[user goal]

Explored context:
[codepaths, files, tests, dependencies, constraints]

Return:
- Proposed sequence
- Ownership boundaries and likely edit surfaces
- Risks and assumptions
- Verification strategy
- Rollback, migration, or rollout concerns
```

## Update Prompt Shape

```text
Update the plan in response to this critique.

Current plan:
[plan]

Critique:
[blocking findings and concerns]

Rules:
- Address each blocking critique directly.
- Preserve good parts of the plan.

Return:
- Revised plan
- Critique resolution notes
- Remaining blockers, if any
```

## Critic Prompt Shape

```text
Critique this plan adversarially before implementation.

Focus on:
- Missed requirements
- Hidden assumptions
- Sequencing and dependency risks
- Security, data, migration, or rollback gaps
- Weak or missing verification
- Scope that should be split or deferred

Return blocking critiques first.
Say clearly if the plan is sufficient to hand to implementation.
```
