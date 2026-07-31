---
name: pr-description
description: Writes concise, evidence-backed pull request descriptions using the Paperbase repository format. Use when creating or updating a PR, drafting a PR body, or when the user asks for a PR description or summary.
---

# Pull Request Description

Write the PR body from the actual diff, commits, tests, and relevant issue or ADR. Never infer verification or behavioural changes without evidence.

## Required structure

```md
## Summary / Problem
[What prompted the change: feature, bug, refactor, or architecture need. Explain why it matters and what changed.]

## Behavioural Changes
[Describe externally observable behaviour before and after. Write “No intended behavioural changes” for a pure refactor.]

## Documentation Changes
[List the user, operator, architecture, domain-language, or developer documentation updated. Write “No documentation changes required” with a brief reason when none are needed.]

## Verification
- `[exact command]` — [result]
```

## Conditional sections

Include only when applicable, in this order between Behavioural Changes and Verification.

### Root Cause

Bug fixes only. State the underlying failure mechanism, not merely the symptom. Explain why existing safeguards or tests did not catch it when known.

### Rollout

Architecture, infrastructure, migration, compatibility, or operational changes only. Cover sequencing, compatibility window, configuration or migration requirements, observability, rollback, and whether a deploy/restart is required. Write “No special rollout required” when the architecture changed but rollout is ordinary.

### Risks / Compatibility

Include only for a meaningful residual risk, backward-compatibility concern, security implication, data migration, or intentionally deferred follow-up. Do not add an empty risk section.

### Screenshots

UI changes only. Include before/after screenshots or say why screenshots are unavailable.

## Rules

- Lead with user or operator impact, not a file list.
- Distinguish intended behaviour from implementation details.
- Use **Root Cause**, not “root cause analysis,” and only for bugs.
- Use exact test commands and outcomes; include pass, fail, and skip counts when available.
- Never write “tests passed” without naming the tests or command.
- Say what was not run and why.
- Mention schema, protocol, dependency, configuration, or deployment changes explicitly when relevant.
- Treat documentation as part of the change: name updated files and what they now explain, or justify why no documentation changed.
- Link issues and ADRs inline where they explain the motivation or decision.
- Keep the body concise; omit empty or irrelevant sections.
- Do not repeat the commit list or narrate every changed file.

## Final check

Before publishing, verify the description matches the current PR diff, the title follows repository conventions, conditional sections are justified, and every verification claim has evidence.
