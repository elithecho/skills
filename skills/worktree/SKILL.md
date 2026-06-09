---
name: worktree
description: Manage Git worktrees with the `wt` CLI, including creating, switching, listing, removing, and moving an active branch into a `.worktrees/` checkout. Use when the user mentions `wt`, worktrees, branch worktrees, or asks to move a branch out of the main checkout.
---

# Worktree

Use the `wt` CLI as the preferred wrapper for common Git worktree operations.

## Quick commands

```bash
wt list
wt add <name>
wt add <name> <branch-or-start-point>
wt switch <name>
wt remove <name>
wt original
wt clean
```

`wt add` creates worktrees under `.worktrees/`:

- `wt add feature-auth` creates `.worktrees/feature-auth` on branch `feature-auth`.
- `wt add hotfix main` creates branch `hotfix` from `main`.
- `wt add file-assets-phase-1 feature/file-assets-phase-1` attaches the existing branch when the worktree name matches the branch basename.

## Move the current branch into a worktree

When the current branch should move out of the main checkout:

1. Inspect state:
   ```bash
   git status --short --branch
   git branch --show-current
   wt list
   ```
2. If there are local changes, stash them with index/untracked state:
   ```bash
   git stash push --include-untracked -m "move <branch> into worktree <timestamp>"
   ```
3. Switch the main checkout to the desired base branch, usually `main`:
   ```bash
   git switch main
   ```
4. Create/attach the branch worktree:
   ```bash
   wt add <worktree-name> <branch>
   ```
5. Restore the stash in the new worktree, then drop it only after a successful apply:
   ```bash
   git -C .worktrees/<worktree-name> stash apply --index stash@{0}
   git stash drop stash@{0}
   ```
6. Verify both checkouts:
   ```bash
   git status --short --branch
   git -C .worktrees/<worktree-name> status --short --branch
   wt list
   ```

## Safety notes

- Do not drop a stash until it has applied cleanly in the destination worktree.
- If a branch is already checked out in another worktree, Git will refuse to attach it; create a new branch from it or switch/remove the existing worktree first.
- Prefer `wt list` before destructive actions so you know which path owns each branch.
