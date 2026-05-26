# Skills

A curated collection of agent skills I created and use.

This repo contains my own skills plus a small curated set of local skills I keep around. Some third-party skills are intentionally **not vendored here**; install them from their original sources instead so they stay properly credited and up to date.

## Install

Install this skill collection with `npx`:

```sh
npx skills@latest add elithecho/skills
```

Or with `bunx`:

```sh
bunx skills@latest add elithecho/skills
```

## Included Skills

| Skill | Description |
|-------|-------------|
| **frontend-component** | Split frontend UI into logical, composable components |
| **orchestrate** | Bounded delegated implementation workflow |
| **plan** | Self-correcting execution plans for risky or cross-component work |
| **ruby-llm** | Ruby LLM integration skill for chat, embeddings, tools, Rails, and more |
| **security-review** | Comprehensive security review on code |
| **worktree** | Manage Git worktrees with the [`wt`](https://github.com/elithecho/wt) CLI |

## My Skills

These are mine / maintained as part of this curated collection:

- [`frontend-component`](./skills/frontend-component/)
- [`plan`](./skills/plan/)
- [`orchestrate`](./skills/orchestrate/)
- [`ruby-llm`](./skills/ruby-llm/)
- [`worktree`](./worktree/) — helper workflows for the [`wt`](https://github.com/elithecho/wt) CLI

## Third-Party Sources

### Matt Pocock skills

Credit: **[Matt Pocock](https://github.com/mattpocock)**

My preferred Matt Pocock skills — install them from the source package:

```sh
npx skills@latest add mattpocock/skills
# or, with bun:
bunx skills@latest add mattpocock/skills
```

Skills sourced from Matt include:

- `grill` — Interviewer-style plan stress-testing
- `grill-with-docs` — Plan stress-testing with domain documentation
- `improve-codebase-architecture` — Architecture deepening and refactoring opportunities
- `to-prd` — Context-to-PRD generation
- `to-issues` — Break plans into grabbable issues
- `tdd` — Red-green-refactor test-driven development
- `diagnose` — Bug diagnosis and performance regression loop
- `triage` — Issue triage state machine
- `write-a-skill` — Create new agent skills with proper structure
- `skill` — Manage local skills
- `skill-creator` — Guide for creating effective agent skills
- `visual-verdict` — Structured visual QA comparisons

### Vercel Labs agent-browser

Source: <https://github.com/vercel-labs/agent-browser>

Install the browser automation CLI:

```sh
npm install -g agent-browser
agent-browser install  # Download Chrome from Chrome for Testing, first time only
```

Install the skill:

```sh
npx skills add vercel-labs/agent-browser
# or, with bun:
bunx skills add vercel-labs/agent-browser
```

### Uncodixfy

Source: <https://github.com/cyxzdev/Uncodixfy>

`uncodixfy` is not vendored in this repo. Use the upstream source above.

### Composio CLI

`composio-cli` is not vendored in this repo. Use Composio's own CLI and documentation for current tool schemas and setup instructions.

### web-clone

`web-clone` is not vendored in this repo. Use the upstream source directly.

## License

The curation and arrangement of this collection is licensed under the [MIT License](./LICENSE) © 2026 Elijah Goh.

Individual skills retain their original licenses where specified.
