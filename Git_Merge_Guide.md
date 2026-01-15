# Git Merge Guide - CLI

Complete guide for merging branches in Git using the command line.

## What is Merging?

**Merging** is the process of integrating changes from one branch into another. It combines the commit histories of two branches into a single unified history.

## Before You Merge - Pre-Merge Checklist

```powershell
# 1. Make sure you're on the correct branch
git branch

# 2. Ensure working directory is clean
git status

# 3. Pull latest changes
git pull origin main

# 4. Check what will be merged
git log main..feature-branch --oneline

# 5. Review differences
git diff main..feature-branch
```

## Basic Merge Commands

### Merge Feature Branch into Main

```powershell
# 1. Switch to the branch you want to merge INTO
git checkout main

# 2. Pull latest changes
git pull origin main

# 3. Merge the feature branch
git merge feature-branch

# 4. Push the merged changes
git push origin main
```

### Merge with No Fast-Forward
```powershell
# Creates a merge commit even if fast-forward is possible
git merge --no-ff feature-branch

# Good for: Preserving branch history
```

### Fast-Forward Merge
```powershell
# Only fast-forward (fails if not possible)
git merge --ff-only feature-branch

# Good for: Linear history when branch hasn't diverged
```

## Merge Strategies

### 1. Fast-Forward Merge (Default when possible)
```
Before:
main:     A---B---C
                   \
feature:            D---E

After:
main:     A---B---C---D---E
```

```powershell
git checkout main
git merge feature-branch
# Result: No merge commit, linear history
```

### 2. Three-Way Merge (When branches diverged)
```
Before:
main:     A---B---C---F
                   \
feature:            D---E

After:
main:     A---B---C---F---M
                   \     /
feature:            D---E
```

```powershell
git checkout main
git merge feature-branch
# Result: Creates merge commit M
```

### 3. Squash Merge (Combine all commits)
```
Before:
feature: D---E---F---G

After (on main):
main: A---B---C---S (S contains all changes from D-G)
```

```powershell
git checkout main
git merge --squash feature-branch
git commit -m "Feature: description of all changes"

# Good for: Cleaner history, many small commits become one
```

### 4. Rebase and Merge (Linear history)
```
Before:
main:     A---B---C---F
                   \
feature:            D---E

After rebase:
main:     A---B---C---F
feature:              \
                       D'---E'

After merge:
main:     A---B---C---F---D'---E'
```

```powershell
# From feature branch
git checkout feature-branch
git rebase main
git checkout main
git merge feature-branch  # Will be fast-forward

# Good for: Clean linear history
```

## Step-by-Step Merge Workflows

### Workflow 1: Standard Feature Branch Merge

```powershell
# Step 1: Ensure your feature branch is up to date
git checkout feature-branch
git pull origin feature-branch

# Step 2: Update main branch
git checkout main
git pull origin main

# Step 3: Merge feature into main
git merge feature-branch

# Step 4: Review the merge
git log --oneline -5

# Step 5: Push to remote
git push origin main

# Step 6: Delete feature branch (optional)
git branch -d feature-branch
git push origin --delete feature-branch
```

### Workflow 2: Merge with Testing First

```powershell
# Step 1: Create test branch
git checkout main
git checkout -b test-merge

# Step 2: Test merge in test branch
git merge feature-branch

# Step 3: Run tests
npm test

# Step 4: If tests pass, do actual merge
git checkout main
git merge feature-branch
git push origin main

# Step 5: Delete test branch
git branch -d test-merge
```

### Workflow 3: Update Feature Branch from Main First

```powershell
# Step 1: Get latest main
git checkout main
git pull origin main

# Step 2: Update feature branch with main's changes
git checkout feature-branch
git merge main
# Or: git rebase main

# Step 3: Resolve any conflicts

# Step 4: Test feature branch
npm test

# Step 5: Merge into main
git checkout main
git merge feature-branch

# Step 6: Push
git push origin main
```

## Handling Merge Conflicts

### When Conflicts Occur

```powershell
# Attempt merge
git merge feature-branch

# Output shows conflicts:
# Auto-merging file.txt
# CONFLICT (content): Merge conflict in file.txt
# Automatic merge failed; fix conflicts and then commit the result.
```

### Resolving Conflicts - Step by Step

```powershell
# 1. See which files have conflicts
git status

# 2. Open conflicted files and look for conflict markers
#    <<<<<<< HEAD
#    Your current branch changes
#    =======
#    Changes from the branch being merged
#    >>>>>>> feature-branch

# 3. Edit the file to resolve conflicts
#    - Keep what you need
#    - Remove conflict markers
#    - Test the resolved code

# 4. Mark as resolved
git add file.txt

# 5. Check all conflicts are resolved
git status

# 6. Complete the merge
git commit -m "Merge feature-branch into main"
# Or just: git commit (uses default merge message)

# 7. Push
git push origin main
```

### Conflict Resolution Tools

```powershell
# Use merge tool
git mergetool

# Accept all changes from your branch
git checkout --ours file.txt

# Accept all changes from their branch
git checkout --theirs file.txt

# View conflict in different ways
git diff
git diff --ours
git diff --theirs

# Abort merge and start over
git merge --abort
```

### Example Conflict Resolution

**Before (conflicted file):**
```javascript
<<<<<<< HEAD
function greet() {
    console.log("Hello World");
}
=======
function greet() {
    console.log("Hi there!");
}
>>>>>>> feature-branch
```

**After (resolved):**
```javascript
function greet() {
    console.log("Hello World"); // Keeping main version
}
```

```powershell
# Mark as resolved
git add file.js
git commit
git push
```

## Common Merge Scenarios

### Scenario 1: Merge with No Conflicts

```powershell
git checkout main
git pull origin main
git merge feature-branch
# Automatic merge successful

git push origin main
```

### Scenario 2: Merge with Conflicts

```powershell
git checkout main
git pull origin main
git merge feature-branch
# CONFLICT appears

# Fix conflicts in files
git status
# Edit conflicted files
git add .
git commit -m "Resolve merge conflicts"
git push origin main
```

### Scenario 3: Merge Multiple Branches

```powershell
git checkout main
git pull origin main

# Merge first feature
git merge feature-1
git push origin main

# Merge second feature
git merge feature-2
git push origin main
```

### Scenario 4: Merge and Squash

```powershell
git checkout main
git pull origin main
git merge --squash feature-branch

# Review staged changes
git status

# Commit with meaningful message
git commit -m "Feature: Add user authentication with email verification"
git push origin main
```

### Scenario 5: Cherry-Pick Instead of Merge

```powershell
# When you only want specific commits, not entire branch
git checkout main

# Pick specific commits
git cherry-pick abc123
git cherry-pick def456

git push origin main
```

## Undoing a Merge

### Before Pushing

```powershell
# Undo merge that hasn't been pushed
git reset --hard HEAD~1

# Or if you know the commit before merge
git reset --hard abc123
```

### After Pushing

```powershell
# Option 1: Revert the merge commit
git revert -m 1 HEAD
git push origin main

# Option 2: Reset and force push (dangerous on shared branches!)
git reset --hard HEAD~1
git push --force origin main
```

### Find the Merge Commit to Undo

```powershell
# Show merge commits
git log --merges --oneline

# Show last merge
git log --merges -1

# Revert specific merge
git revert -m 1 abc123
```

## Merge vs Rebase

### When to Use Merge

✅ **Use Merge When:**
- Working on public/shared branches
- You want to preserve complete history
- Multiple people are working on the branch
- Feature development is complete
- You want to see when features were integrated

```powershell
git merge feature-branch
```

### When to Use Rebase

✅ **Use Rebase When:**
- Working on private/local branches
- You want clean linear history
- Updating feature branch with main
- Before creating pull request
- You haven't pushed the branch yet

```powershell
git rebase main
```

**⚠️ Never rebase public branches that others are using!**

## GitHub Pull Request Merge

### Merge via GitHub UI

Three options available on GitHub:

**1. Create a Merge Commit**
```
Preserves all commits from feature branch
Creates merge commit
```

**2. Squash and Merge**
```
Combines all commits into one
Cleaner main branch history
```

**3. Rebase and Merge**
```
Adds commits individually without merge commit
Linear history
```

### Merge via GitHub CLI

```powershell
# Install GitHub CLI
# https://cli.github.com/

# List PRs
gh pr list

# Merge PR with merge commit
gh pr merge 123 --merge

# Merge PR with squash
gh pr merge 123 --squash

# Merge PR with rebase
gh pr merge 123 --rebase

# Merge and delete branch
gh pr merge 123 --squash --delete-branch

# Auto-merge when checks pass
gh pr merge 123 --auto --squash
```

## Best Practices

### Before Merging

1. **Test thoroughly** - Run all tests on feature branch
2. **Code review** - Get PR approved
3. **Update from main** - Ensure feature branch is current
4. **CI/CD passes** - All checks must be green
5. **Resolve conflicts** - Fix conflicts before merging
6. **Clean commits** - Squash messy commits if needed

### During Merging

1. **Use descriptive messages** - Explain what's being merged
2. **Review changes** - Check diff before merging
3. **Merge to correct branch** - Double-check target branch
4. **One feature at a time** - Don't mix unrelated changes

### After Merging

1. **Delete feature branch** - Clean up merged branches
2. **Tag releases** - Tag important merges
3. **Update team** - Notify team of significant merges
4. **Deploy** - Follow deployment process
5. **Monitor** - Watch for issues post-merge

## Merge Commit Messages

### Good Merge Messages

```
✅ Merge branch 'feature/user-auth' into main
✅ Merge pull request #123 from user/feature-branch
✅ Merge feature: Add payment processing
✅ Merge bugfix: Fix login validation error
```

### Detailed Merge Message Template

```
Merge branch 'feature/user-authentication'

This feature adds:
- Email/password authentication
- JWT token handling
- Password reset functionality

Closes #456
Reviewed-by: @reviewer
Tested-by: QA Team
```

## Troubleshooting

### Merge Fails - Unrelated Histories

```powershell
# Error: refusing to merge unrelated histories
# Solution:
git merge feature-branch --allow-unrelated-histories
```

### Merge Creates Unwanted Files

```powershell
# Check what's being merged
git diff main..feature-branch --name-only

# If wrong files are included:
git merge --abort
# Review and clean feature branch first
```

### Accidentally Merged Wrong Branch

```powershell
# Before push:
git reset --hard HEAD~1

# After push (creates revert commit):
git revert -m 1 HEAD
git push
```

### Can't Merge - Uncommitted Changes

```powershell
# Option 1: Commit changes
git add .
git commit -m "WIP: Save work"

# Option 2: Stash changes
git stash
git merge feature-branch
git stash pop

# Option 3: Discard changes
git reset --hard
git merge feature-branch
```

### Merge Conflict in Binary File

```powershell
# Choose version from main
git checkout --ours file.bin
git add file.bin

# Choose version from feature
git checkout --theirs file.bin
git add file.bin

git commit
```

## Advanced Merge Techniques

### Octopus Merge (Multiple Branches)

```powershell
# Merge multiple branches at once
git merge branch1 branch2 branch3

# Only works if no conflicts
```

### Subtree Merge

```powershell
# Merge external project as subdirectory
git merge -s subtree other-branch
```

### Custom Merge Strategy

```powershell
# Use specific merge strategy
git merge -s recursive -X ours feature-branch
git merge -s recursive -X theirs feature-branch

# Strategies: recursive, resolve, octopus, ours, subtree
```

## Complete Merge Checklist

```markdown
### Pre-Merge Checklist
- [ ] Feature branch is complete
- [ ] All tests passing
- [ ] Code reviewed and approved
- [ ] CI/CD checks passed
- [ ] Branch updated with latest main
- [ ] Conflicts resolved
- [ ] Documentation updated
- [ ] Changelog updated

### Merge Process
- [ ] On correct target branch (main)
- [ ] Latest changes pulled
- [ ] Merge command executed
- [ ] Conflicts resolved (if any)
- [ ] Tests run after merge
- [ ] Merge commit created
- [ ] Changes pushed to remote

### Post-Merge
- [ ] Feature branch deleted
- [ ] Team notified
- [ ] Deployment triggered (if applicable)
- [ ] Monitoring set up
- [ ] Related tickets closed
```

## Quick Reference

### Basic Merge
```powershell
git checkout main
git pull
git merge feature-branch
git push
```

### Merge with Squash
```powershell
git merge --squash feature-branch
git commit -m "Feature: description"
git push
```

### Abort Merge
```powershell
git merge --abort
```

### Resolve Conflicts
```powershell
# Edit files
git add .
git commit
git push
```

### Undo Merge
```powershell
# Before push
git reset --hard HEAD~1

# After push
git revert -m 1 HEAD
git push
```

## Common Mistakes to Avoid

❌ Merging into wrong branch
❌ Not pulling latest changes first
❌ Ignoring merge conflicts
❌ Force pushing after merge
❌ Not testing after merge
❌ Merging broken code
❌ Not deleting merged branches
❌ Rewriting history of public branches
❌ Mixing unrelated features
❌ Not communicating large merges

## Merge Strategy Decision Tree

```
Is branch public/shared?
├─ Yes → Use MERGE
└─ No → Continue

Do you want to preserve all commits?
├─ Yes → Use MERGE
└─ No → Continue

Is history messy with many small commits?
├─ Yes → Use SQUASH MERGE
└─ No → Continue

Do you want linear history?
├─ Yes → Use REBASE then MERGE
└─ No → Use standard MERGE
```

## Resources

- [Git Merge Documentation](https://git-scm.com/docs/git-merge)
- [Atlassian Merge Tutorial](https://www.atlassian.com/git/tutorials/using-branches/git-merge)
- [GitHub Pull Request Merge](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/merging-a-pull-request)
- [Git Merge Strategies](https://git-scm.com/docs/merge-strategies)

## Summary

**Simple Merge:**
```powershell
git checkout main
git pull
git merge feature-branch
git push
```

**With Conflict Resolution:**
```powershell
git merge feature-branch
# Fix conflicts
git add .
git commit
git push
```

**Squash for Clean History:**
```powershell
git merge --squash feature-branch
git commit -m "Descriptive message"
git push
```

Remember: **Always test after merging and communicate with your team about significant merges!**
