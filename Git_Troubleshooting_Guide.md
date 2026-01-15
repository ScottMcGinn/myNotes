# Git Troubleshooting and Diagnostics Guide

This guide covers common Git issues and diagnostic commands to help identify and resolve problems.

## Diagnostic Commands

### Check Git Version
```powershell
git --version
```
Displays the installed Git version. Useful for verifying installation and compatibility.

### Check Repository Status
```powershell
git status
```
Shows:
- Current branch
- Modified files
- Staged files
- Untracked files
- Whether your branch is ahead/behind the remote

### View Detailed Status
```powershell
git status -v
```
Shows more verbose output including the diff of staged changes.

### Check Remote Connections
```powershell
# List all remotes
git remote -v

# Show detailed info about a remote
git remote show origin

# Test connection to remote
git ls-remote origin
```

### View Repository Configuration
```powershell
# View all configuration
git config --list

# View global config
git config --global --list

# View local repository config
git config --local --list

# Check specific setting
git config user.name
git config user.email
```

### View Commit History
```powershell
# Standard log
git log

# One line per commit
git log --oneline

# Last 5 commits
git log -5

# Graph view
git log --oneline --graph --all

# Show changes in commits
git log -p

# Commits by specific author
git log --author="YourName"

# Commits in date range
git log --since="2 weeks ago" --until="yesterday"
```

### View File Changes
```powershell
# Changes in working directory (not staged)
git diff

# Changes that are staged
git diff --staged

# Changes in a specific file
git diff filename.txt

# Compare branches
git diff branch1..branch2

# Show file at specific commit
git show commit-hash:path/to/file
```

### Check What Will Be Pushed
```powershell
# See commits that will be pushed
git log origin/master..HEAD

# See differences that will be pushed
git diff origin/master..HEAD
```

### Check Branch Information
```powershell
# List all local branches
git branch

# List all branches (local and remote)
git branch -a

# List remote branches only
git branch -r

# Show branch tracking info
git branch -vv

# Show merged branches
git branch --merged

# Show unmerged branches
git branch --no-merged
```

### View Reflog (Command History)
```powershell
# Shows history of HEAD movements
git reflog

# Shows last 10 actions
git reflog -10
```

## Common Issues and Solutions

### Issue: "fatal: not a git repository"

**Diagnosis:**
```powershell
pwd  # Check you're in the right directory
ls -Force  # Look for .git folder
```

**Solution:**
```powershell
# Initialize new repository
git init

# Or clone existing repository
git clone https://github.com/user/repo.git
```

---

### Issue: Authentication Failed

**Diagnosis:**
```powershell
# Check remote URL
git remote -v

# Test SSH connection (if using SSH)
ssh -T git@github.com
```

**Solutions:**

**For HTTPS:**
```powershell
# Use Personal Access Token instead of password
# Generate at: GitHub Settings → Developer Settings → Personal Access Tokens

# Update remote URL to use token
git remote set-url origin https://YOUR-TOKEN@github.com/user/repo.git
```

**For SSH:**
```powershell
# Generate SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy public key to clipboard (Windows)
Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard

# Add to GitHub: Settings → SSH and GPG keys → New SSH key
```

---

### Issue: Cannot Push - "Updates were rejected"

**Diagnosis:**
```powershell
# Check if remote has changes you don't have
git fetch
git status
```

**Solution:**
```powershell
# Pull changes first
git pull --rebase
# Or
git pull

# Then push
git push
```

---

### Issue: Merge Conflicts

**Diagnosis:**
```powershell
# See which files have conflicts
git status

# View the conflict
git diff
```

**Solution:**
```powershell
# 1. Open conflicted files and resolve manually
# Look for conflict markers: <<<<<<<, =======, >>>>>>>

# 2. Stage resolved files
git add filename

# 3. Complete the merge
git commit

# Or abort the merge
git merge --abort
```

---

### Issue: Accidentally Committed Wrong Files

**Diagnosis:**
```powershell
# Check what was committed
git log -1
git show HEAD
```

**Solutions:**

**Undo last commit (keep changes):**
```powershell
git reset --soft HEAD~1
```

**Undo last commit (discard changes):**
```powershell
git reset --hard HEAD~1
```
⚠️ **Warning:** This permanently deletes changes!

**Remove file from last commit:**
```powershell
git reset HEAD~1 path/to/file
git commit --amend
```

---

### Issue: Need to Undo Changes to a File

**Diagnosis:**
```powershell
# See what changed
git diff filename
```

**Solutions:**

**Discard unstaged changes:**
```powershell
git restore filename
# Or (older Git versions)
git checkout -- filename
```

**Unstage a staged file:**
```powershell
git restore --staged filename
# Or (older Git versions)
git reset HEAD filename
```

**Revert to specific commit:**
```powershell
git checkout commit-hash -- filename
```

---

### Issue: Detached HEAD State

**Diagnosis:**
```powershell
git status
# Will show: "HEAD detached at commit-hash"

git branch
# Will show: "* (HEAD detached at commit-hash)"
```

**Solution:**
```powershell
# Return to a branch
git checkout master

# Or create a new branch from current position
git checkout -b new-branch-name
```

---

### Issue: Large Files / Slow Push

**Diagnosis:**
```powershell
# Find large files in repository
git rev-list --objects --all | `
    git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' | `
    Where-Object { $_ -match '^blob' } | `
    Sort-Object { [int]($_ -split ' ')[2] } -Descending | `
    Select-Object -First 10

# Check repository size
git count-objects -vH
```

**Solution:**
```powershell
# Add large files to .gitignore
# Remove large file from history (careful!)
git filter-branch --tree-filter 'rm -f path/to/largefile' HEAD

# Or use BFG Repo-Cleaner (recommended)
# Download from: https://rtyley.github.io/bfg-repo-cleaner/
```

---

### Issue: Wrong Commit Message

**Diagnosis:**
```powershell
git log -1
```

**Solution:**

**Fix last commit message:**
```powershell
git commit --amend -m "New commit message"
```

**Fix older commit message:**
```powershell
# Rebase interactive
git rebase -i HEAD~3  # For last 3 commits

# Change 'pick' to 'reword' for the commit you want to change
# Save and close, then edit the commit message
```

---

### Issue: Untracked Files Not Being Ignored

**Diagnosis:**
```powershell
# Check .gitignore exists
cat .gitignore

# Test if file matches .gitignore pattern
git check-ignore -v filename
```

**Solution:**
```powershell
# Files already tracked won't be ignored
# Remove from tracking but keep file
git rm --cached filename

# Remove directory from tracking
git rm --cached -r directory/

# Commit the removal
git commit -m "Remove tracked files now in .gitignore"
```

---

### Issue: Lost Commits After Reset

**Diagnosis:**
```powershell
# View command history
git reflog
```

**Solution:**
```powershell
# Find the lost commit in reflog
git reflog

# Recover the commit
git cherry-pick commit-hash

# Or reset to that commit
git reset --hard commit-hash
```

---

### Issue: Need to Work on Multiple Features Simultaneously

**Solution:**
```powershell
# Save current work without committing
git stash

# List stashed changes
git stash list

# Apply most recent stash
git stash apply

# Apply and remove most recent stash
git stash pop

# Apply specific stash
git stash apply stash@{2}

# Create named stash
git stash save "Work in progress on feature X"

# Delete specific stash
git stash drop stash@{0}

# Clear all stashes
git stash clear
```

---

### Issue: Clone is Very Slow

**Solution:**
```powershell
# Shallow clone (only recent history)
git clone --depth 1 https://github.com/user/repo.git

# Clone specific branch only
git clone --branch branch-name --single-branch https://github.com/user/repo.git
```

---

## Advanced Diagnostic Commands

### Check for Corrupted Objects
```powershell
git fsck --full
```

### Cleanup and Optimize Repository
```powershell
# Remove unnecessary files and optimize
git gc

# Aggressive cleanup
git gc --aggressive --prune=now
```

### Verify Connectivity
```powershell
# Check all objects are reachable
git fsck --connectivity-only
```

### Show Ignored Files
```powershell
# List all ignored files
git status --ignored

# Show only ignored files
git clean -ndX
```

### Find Who Changed a Line
```powershell
# Show who last modified each line
git blame filename

# Blame specific lines
git blame -L 10,20 filename
```

### Search Commit Messages
```powershell
# Search for keyword in commit messages
git log --grep="keyword"

# Search for keyword in code changes
git log -S "keyword"
```

### Compare File Between Branches
```powershell
git diff branch1:path/file branch2:path/file
```

### List Files in a Commit
```powershell
git show --name-only commit-hash
```

### Recover Deleted File
```powershell
# Find when file was deleted
git log --all --full-history -- path/to/file

# Restore from before deletion
git checkout commit-hash^ -- path/to/file
```

## Useful Aliases

Add these to your Git config for faster diagnostics:

```powershell
# Set useful aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual 'log --oneline --graph --all --decorate'
git config --global alias.aliases 'config --get-regexp alias'
```

## Emergency Commands

### Completely Reset to Match Remote
```powershell
git fetch origin
git reset --hard origin/master
```
⚠️ **Warning:** This deletes all local changes!

### Start Fresh (Nuclear Option)
```powershell
# Backup first!
rm -rf .git
git init
git remote add origin https://github.com/user/repo.git
git fetch
git reset --hard origin/master
```

### Undo Git Init
```powershell
rm -rf .git
```

## Best Practices for Avoiding Issues

1. **Commit often** - Small, frequent commits are easier to manage
2. **Pull before push** - Always pull latest changes before pushing
3. **Use branches** - Keep master/main stable, work in feature branches
4. **Write clear commit messages** - Explain what and why, not how
5. **Review before committing** - Use `git diff` and `git status`
6. **Use .gitignore** - Don't commit sensitive or generated files
7. **Test before merging** - Ensure code works before merging to main branch

## Getting Help

```powershell
# General help
git help

# Help for specific command
git help commit
git commit --help

# Quick reference
git command -h
```

## Additional Resources

- Official Git Documentation: https://git-scm.com/doc
- GitHub Guides: https://guides.github.com/
- Interactive Git Tutorial: https://learngitbranching.js.org/
