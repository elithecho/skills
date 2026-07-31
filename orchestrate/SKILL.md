---
name: orchestrate
description: Runs the bounded development orchestration workflow for implementation tasks that need exploration, delegated coding, adversarial review, and verification without a separate architecture planning gate. Use when a task mentions orchestrate, bounded delegated implementation, closed review loop, staged execution, or when AGENTS.md should stay light and execution workflow should be loaded on demand.
---

# Orchestrate

## Purpose

Use this skill for bounded development work that is more than a tiny local edit but does not need a separate architecture planning gate.

Bounded orchestration work usually has:

- A clear user goal
- Some uncertainty requiring codebase exploration
- A likely edit surface of a few files
- Moderate regression risk
- Concrete verification criteria

Do not use this skill for trivial single-file edits. Do not use it for architecture-level changes, broad migrations, API redesigns, or unclear multi-system changes; those need the `plan` skill first.

## Operating Limits

- Use at most **two repair rounds** in the review loop. After the second failed review, close the loop and move to verification — do not stop, re-plan, or re-review. Carry forward any unresolved non-blocking findings as deferred follow-ups.
- Only critical/high findings block automatically. Medium findings require an explicit ship-now versus follow-up judgment.
- Do not launch a full reviewer for a trivial, mechanically verifiable correction. Fix it directly and verify.

## Workflow

1. Classify and state that the task needs bounded orchestration.
   - Give the user a brief reason.
   - If exploration proves the task is trivial, continue locally.
   - If exploration proves the task needs architecture planning, stop this workflow and load the `plan` skill.

2. Explore before execution.
   - Map the relevant codepath, owners, tests, and likely edit surface.
   - Prefer a bounded `explore` subagent for discovery when available.
   - If behavior is underprotected or the user asks for test-first work, load the `tdd` skill before implementation.
   - Keep local reads focused on integration and risk judgment.

3. Delegate the first implementation pass.
   - Send the explored context to an `executor`.
   - Give explicit ownership: files, modules, or responsibility.
   - Tell the executor they are not alone in the codebase and must not revert unrelated edits.
   - Ask for changed files, verification run, and any residual risk in the final report.

4. Run a closed review loop.
   - Send the implementation artifact to `code-reviewer`.
   - Ask for adversarial review focused on correctness, regressions, security, test gaps, edge cases, hidden assumptions, and missed requirements.
   - Send blocking findings back to the original executor for fixes.
   - Use at most **two repair rounds**. After the second failed review, close the loop and move to verification — do not re-plan or re-review. Carry forward unresolved non-blocking findings as deferred follow-ups.
   - Only critical/high findings block automatically. Medium findings require an explicit ship-now versus follow-up judgment.
   - If a trivial, mechanically verifiable fix surfaces during review, fix it directly rather than launching a full re-review cycle.

5. Verify after review is clean.
   - Run local verification or delegate to `verifier` or `test-engineer` only after implementation exists.
   - Use the checks that prove the claim: tests, typecheck, lint, build, static analysis, or targeted runtime checks.
   - If verification fails, iterate before reporting completion.

6. Final report.
   - List changed files.
   - Summarize the simplifications or implementation made.
   - Report verification evidence.
   - Call out remaining risks or skipped checks.

## Executor Prompt Shape

```text
You are implementing a bounded development task as the executor.

Goal:
[user goal]

Explored context:
[files, symbols, tests, constraints]

Ownership:
[files/modules/responsibility]

Rules:
- You are not alone in the codebase; do not revert unrelated edits.
- Follow existing patterns and keep the diff small.
- Use the `tdd` skill for red-green-refactor when behavior needs to be locked before implementation.
- No new dependencies unless explicitly requested.
- Verify the change with the narrowest meaningful checks.

Return:
- Changed files
- What changed
- Verification run and result
- Remaining risks
```

## Reviewer Prompt Shape

```text
Review this bounded task implementation adversarially.

Focus on:
- Correctness and regressions
- Security and trust boundaries
- Test gaps and missed edge cases
- Hidden assumptions
- Whether it satisfies the original request

Return blocking findings first with file and line references.
Say clearly if there are no blocking findings.
```
