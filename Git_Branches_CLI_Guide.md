# Git Branches - CLI Guide

Complete guide for working with Git branches using the command line.

## What is a Branch?

A **branch** is a parallel version of your repository. It allows you to work on features, fixes, or experiments without affecting the main codebase.

## Basic Branch Commands

### Check Current Branch
```powershell
# Show current branch
git branch

# Show current branch with more info
git status

# Show all branches (local and remote)
git branch -a

# Show only remote branches
git branch -r
```

## Creating Branches

### Create a New Branch
```powershell
# Create branch (but stay on current branch)
git branch feature-name

# Create branch and switch to it
git checkout -b feature-name

# Modern way (Git 2.23+)
git switch -c feature-name

# Create from specific commit
git checkout -b feature-name abc123

# Create from specific branch
git checkout -b new-feature main
```

### Branch from Current Location
```powershell
# Create branch from where you are now
git checkout -b my-feature

# This creates a new branch from HEAD (current position)
```

### Branch from Specific Commit
```powershell
# View commit history
git log --oneline

# Create branch from specific commit hash
git checkout -b hotfix a1b2c3d

# Create branch from tag
git checkout -b release-branch v1.0.0
```

### Branch from Remote Branch
```powershell
# Fetch latest from remote
git fetch origin

# Create local branch tracking remote branch
git checkout -b feature-name origin/feature-name

# Shorthand (if branch doesn't exist locally)
git checkout feature-name

# Modern way
git switch feature-name
```

## Switching Branches

### Switch to Existing Branch
```powershell
# Old way
git checkout branch-name

# Modern way (Git 2.23+)
git switch branch-name

# Switch to previous branch
git checkout -

# Switch and create if doesn't exist
git checkout -b branch-name
git switch -c branch-name
```

### Switch with Uncommitted Changes
```powershell
# Stash changes first
git stash
git checkout other-branch

# Later, restore stashed changes
git stash pop

# Or switch with changes (if no conflicts)
git checkout other-branch  # Git will move changes with you
```

## Pushing Branches

### Push New Branch to Remote
```powershell
# Push and set upstream
git push -u origin feature-name

# Long form
git push --set-upstream origin feature-name

# After first push, just use
git push
```

### Push with Different Name
```powershell
# Push local branch to remote with different name
git push origin local-branch:remote-branch

# Example
git push origin feature:feature-v2
```

### Push All Branches
```powershell
# Push all local branches
git push --all origin
```

## Renaming Branches

### Rename Current Branch
```powershell
# Rename the branch you're on
git branch -m new-name

# Long form
git branch --move new-name
```

### Rename Other Branch
```powershell
# Rename branch you're not on
git branch -m old-name new-name
```

### Rename and Update Remote
```powershell
# 1. Rename local branch
git branch -m old-name new-name

# 2. Delete old remote branch
git push origin --delete old-name

# 3. Push new branch and set upstream
git push -u origin new-name
```

## Deleting Branches

### Delete Local Branch
```powershell
# Delete merged branch
git branch -d branch-name

# Force delete (even if not merged)
git branch -D branch-name

# Delete multiple branches
git branch -d branch1 branch2 branch3
```

### Delete Remote Branch
```powershell
# Delete remote branch
git push origin --delete branch-name

# Alternative syntax
git push origin :branch-name
```

### Delete All Merged Branches
```powershell
# List merged branches
git branch --merged

# Delete all merged branches except main/master
git branch --merged | grep -v "main\|master" | xargs git branch -d
```

## Viewing Branches

### List Branches
```powershell
# List local branches
git branch

# List with last commit
git branch -v

# List with tracking info
git branch -vv

# List all (local + remote)
git branch -a

# List only remote branches
git branch -r

# List merged branches
git branch --merged

# List unmerged branches
git branch --no-merged
```

### Show Branch Details
```powershell
# Show commits on current branch
git log

# Show commits unique to this branch
git log main..feature-branch

# Show branch graph
git log --oneline --graph --all

# Show who created the branch
git log --all --format='%h %an %s' feature-branch
```

## Tracking Remote Branches

### Set Upstream Branch
```powershell
# Set upstream for current branch
git branch --set-upstream-to=origin/feature-name

# Short form
git branch -u origin/feature-name

# Set when pushing
git push -u origin feature-name
```

### Check Tracking Status
```powershell
# Show tracking branches
git branch -vv

# Show remote details
git remote show origin
```

### Unset Upstream
```powershell
# Remove upstream tracking
git branch --unset-upstream
```

## Complete Workflow Examples

### Example 1: Create Feature Branch
```powershell
# 1. Make sure you're on main and up to date
git checkout main
git pull origin main

# 2. Create and switch to new branch
git checkout -b feature/user-authentication

# 3. Make changes and commit
# ... edit files ...
git add .
git commit -m "Add user authentication"

# 4. Push to remote
git push -u origin feature/user-authentication

# 5. Create pull request on GitHub
# (Done via GitHub web UI or gh CLI)
```

### Example 2: Work on Existing Remote Branch
```powershell
# 1. Fetch latest branches
git fetch origin

# 2. Checkout remote branch
git checkout feature/new-feature

# 3. Make changes
# ... edit files ...
git add .
git commit -m "Update feature"

# 4. Push changes
git push
```

### Example 3: Create Hotfix Branch
```powershell
# 1. Branch from main
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug

# 2. Fix and commit
# ... fix bug ...
git add .
git commit -m "Fix critical bug"

# 3. Push
git push -u origin hotfix/critical-bug

# 4. Create PR to main
# 5. After merge, delete branch
git checkout main
git pull
git branch -d hotfix/critical-bug
git push origin --delete hotfix/critical-bug
```

### Example 4: Update Branch with Latest Main
```powershell
# On your feature branch
git checkout feature-branch

# Fetch latest
git fetch origin

# Option 1: Merge main into feature
git merge origin/main

# Option 2: Rebase feature on main (cleaner history)
git rebase origin/main

# If conflicts, resolve them, then
git add .
git rebase --continue

# Force push if you rebased (rewrites history)
git push --force-with-lease
```

### Example 5: Create Branch from Tag
```powershell
# List tags
git tag

# Create branch from tag
git checkout -b release/v1.0.1 v1.0.0

# Make changes
git add .
git commit -m "Patch for v1.0.1"

# Push
git push -u origin release/v1.0.1
```

## Branch Naming Conventions

### Common Patterns
```
feature/description      - New features
bugfix/description       - Bug fixes
hotfix/description       - Urgent fixes
release/version          - Release branches
docs/description         - Documentation
refactor/description     - Code refactoring
test/description         - Testing changes
chore/description        - Maintenance tasks
```

### Examples
```
feature/user-login
feature/payment-integration
bugfix/login-error
hotfix/security-patch
release/v1.2.0
docs/api-documentation
refactor/database-layer
test/add-unit-tests
chore/update-dependencies
```

### Best Practices
- Use lowercase
- Use hyphens, not spaces
- Be descriptive but concise
- Include ticket number if applicable: `feature/ABC-123-user-login`
- Keep under 50 characters

## Comparing Branches

### Compare Branches
```powershell
# Show commits in feature not in main
git log main..feature-branch

# Show commits in main not in feature
git log feature-branch..main

# Show difference between branches
git diff main..feature-branch

# Show files changed between branches
git diff --name-only main..feature-branch

# Show stats
git diff --stat main..feature-branch
```

### Find Common Ancestor
```powershell
# Find merge base
git merge-base main feature-branch

# Show when branches diverged
git log --oneline $(git merge-base main feature-branch)..feature-branch
```

## Recovering Branches

### Recover Deleted Branch
```powershell
# Find the commit hash of the deleted branch
git reflog

# Create branch from that commit
git checkout -b recovered-branch abc123
```

### List Recently Deleted Branches
```powershell
# Show reflog
git reflog

# Find deleted branch commits
git reflog | grep "moving from"
```

## Syncing with Remote

### Update Local Branch List
```powershell
# Fetch and prune deleted remote branches
git fetch --prune

# Or
git fetch -p

# Remove tracking branches that are gone
git remote prune origin
```

### Pull Latest Changes
```powershell
# Pull changes to current branch
git pull

# Pull with rebase (cleaner history)
git pull --rebase

# Pull specific branch
git pull origin feature-branch
```

## Advanced Branch Operations

### Cherry-Pick from Another Branch
```powershell
# Pick specific commit from another branch
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456

# Cherry-pick range of commits
git cherry-pick abc123..def456
```

### Merge Branch
```powershell
# Merge feature into current branch
git merge feature-branch

# Merge with no fast-forward (creates merge commit)
git merge --no-ff feature-branch

# Merge and squash commits
git merge --squash feature-branch
```

### Rebase Branch
```powershell
# Rebase current branch onto main
git rebase main

# Interactive rebase (edit commits)
git rebase -i main

# Abort rebase if issues
git rebase --abort

# Continue after resolving conflicts
git rebase --continue
```

## Troubleshooting

### Branch Already Exists
```powershell
# Error: branch already exists
# Solution 1: Use different name
git checkout -b feature-v2

# Solution 2: Delete old branch first
git branch -d old-branch
git checkout -b old-branch

# Solution 3: Force create (overwrites)
git checkout -B branch-name
```

### Can't Switch Due to Uncommitted Changes
```powershell
# Option 1: Commit changes
git add .
git commit -m "WIP: work in progress"

# Option 2: Stash changes
git stash
git checkout other-branch
git stash pop

# Option 3: Discard changes (careful!)
git checkout -- .
git checkout other-branch
```

### Branch Not Showing Up
```powershell
# Fetch latest
git fetch origin

# Check remote branches
git branch -r

# Checkout if exists on remote
git checkout branch-name
```

### Upstream Not Set
```powershell
# Error: no upstream branch
# Solution: Set upstream when pushing
git push -u origin branch-name

# Or set upstream separately
git branch --set-upstream-to=origin/branch-name
```

## Quick Reference

### Create & Switch
```powershell
git checkout -b new-branch          # Create and switch
git switch -c new-branch            # Modern way
```

### View
```powershell
git branch                          # List local
git branch -a                       # List all
git branch -v                       # With last commit
```

### Push
```powershell
git push -u origin branch-name      # First push
git push                            # Subsequent pushes
```

### Delete
```powershell
git branch -d branch-name           # Delete local
git push origin --delete branch     # Delete remote
```

### Rename
```powershell
git branch -m new-name              # Rename current
git branch -m old new               # Rename other
```

### Update
```powershell
git fetch origin                    # Fetch updates
git pull                            # Pull changes
git merge main                      # Merge main
```

## Using GitHub CLI

### Create Branch with gh
```powershell
# Install GitHub CLI
# https://cli.github.com/

# Create branch and PR
gh pr create --base main --head feature-branch

# Checkout PR branch
gh pr checkout 123

# List branches
gh repo view --web
```

## Tips & Best Practices

1. **Always branch from main** - Keep feature branches up to date
2. **Pull before creating** - Get latest changes first
3. **Descriptive names** - Use clear, meaningful branch names
4. **Small branches** - Keep changes focused and reviewable
5. **Delete after merge** - Clean up merged branches
6. **Regular commits** - Commit often with clear messages
7. **Push regularly** - Don't lose work, push to remote
8. **Stay updated** - Regularly merge/rebase from main
9. **Test before PR** - Ensure code works before requesting review
10. **One feature per branch** - Don't mix unrelated changes

## Common Mistakes to Avoid

❌ Creating branches with spaces in names
❌ Branching from wrong branch
❌ Not pulling latest before creating branch
❌ Mixing features in one branch
❌ Not deleting merged branches
❌ Force pushing to shared branches
❌ Creating too many stale branches
❌ Not setting upstream on first push
❌ Working directly on main
❌ Using vague branch names

## Resources

- [Git Branching Documentation](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)
- [GitHub Flow](https://docs.github.com/en/get-started/quickstart/github-flow)
- [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials/using-branches)
- [Git Branch Command Reference](https://git-scm.com/docs/git-branch)
