# pr-shadow

Maintain a shadow branch for GitHub PRs that never requires force-pushing.
This addresses pain points related to GitHub's branch-centric workflow: <https://maskray.me/blog/2023-09-09-reflections-on-llvm-switch-to-github-pull-requests#patch-evolution>

## Problem

When working on PRs, you often rebase, amend, and rewrite history locally. This requires force-pushing to update the PR branch, which loses review context and makes it hard for reviewers to see incremental changes.

## Solution

pr-shadow maintains a separate PR branch that only receives commits (never force-pushed). You work freely on your local branch (rebase, amend, squash), then sync to the PR branch using `git commit-tree` to create a commit with the same tree but parented to the previous PR HEAD.

```
Local branch (feature)     PR branch (pr/feature)
        A                         A
        |                         |
        B (amend)                 C1 "Fix bug"
        |                         |
        C (rebase)                C2 "Address review"
```

Reviewers see clean diffs between C1 and C2, even though the underlying commits were rewritten.

When a rebase is detected (`merge-base` with main/master changed), the new PR commit is created as a merge commit with the new merge-base as the second parent.
GitHub displays these as "condensed" merges, preserving the diff view for reviewers.

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
prs push --force "Rewrite"  # Force push if remote diverged

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

For branches matching `user/<username>/<feature>` or `users/<name>/<feature>`:

- Local branch: `user/<username>/feature`
- PR branch: `user/<username>/pr/feature` (pushed to `origin`)
- Creates PR from `user/<username>/pr/feature` to `main` in the same repo

This is useful for repos where contributors push branches directly instead of using forks.
GitHub Enterprise support**: Host is auto-detected from repository URL

## Requirements

- Ruby
- [gh](https://cli.github.com/) (GitHub CLI)
