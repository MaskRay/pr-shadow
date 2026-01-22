# pr-shadow

Maintain a shadow branch for GitHub PRs that never requires force-pushing.

## Problem

When working on PRs, you often rebase, amend, and rewrite history locally. This requires force-pushing to update the PR branch, which loses review context and makes it hard for reviewers to see incremental changes.

## Solution

prs maintains a separate PR branch that only receives commits (never force-pushed). You work freely on your local branch (rebase, amend, squash), then sync to the PR branch with a commit that captures the new state.

```
Local branch (feature)     PR branch (pr/feature)
        A                         A
        |                         |
        B (amend)                 C1 "Fix bug"
        |                         |
        C (rebase)                C2 "Address review"
```

Reviewers see clean diffs between C1 and C2, even though the underlying commits were rewritten.

## Usage

```bash
# Initialize and create PR
git checkout feature
prs init              # Creates pr/feature, pushes, opens PR
prs init --draft      # Same but creates draft PR

# Work locally
git rebase main
git commit --amend
# etc.

# Sync to PR (message required)
prs push "Fix bug"

# Update PR title/body from local commit message
prs desc

# Run gh commands on the PR
prs gh view
prs gh checks
prs gh edit --add-label bug

# Check status
prs status
```

## Workflows

prs supports two workflows, auto-detected by branch name:

### Fork-based (default)

For branches like `feature`, `fix-bug`, etc:

- PR branch: `pr/<branch>` on your fork
- Creates PR from `YourFork:pr/branch` to `origin:main`

### Same-repo

For branches matching `user/<name>/<feature>` or `users/<name>/<feature>`:

- Local branch: `user/maskray/feature`
- PR branch: `user/maskray/pr/feature` (pushed to `origin`)
- Creates PR from `user/maskray/pr/feature` to `main` in the same repo

This is useful for monorepos where contributors push branches directly instead of using forks.

## Requirements

- Ruby
- [gh](https://cli.github.com/) (GitHub CLI)

## How it works

`prs push` creates a commit using `git commit-tree`:
- Parent: previous PR branch HEAD
- Tree: matches local branch exactly

When a rebase is detected (merge-base with `origin/main` changed), a second parent is added linking to the local HEAD. GitHub shows these as "condensed" merge commits.
