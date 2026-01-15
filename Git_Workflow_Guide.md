# Git Workflow Guide - Pushing to GitHub

This guide walks through the steps to initialize a Git repository and push it to GitHub.

## Prerequisites

- Git installed on your system
- A GitHub account
- A repository created on GitHub (e.g., https://github.com/ScottMcGinn/myNotes)

## Step 1: Check if Git Repository Exists

```powershell
git status
```

**What it does:** Checks if the current directory is already a Git repository.

**Expected output:** If not initialized, you'll see:
```
fatal: not a git repository (or any of the parent directories): .git
```

## Step 2: Initialize a New Git Repository

```powershell
git init
```

**What it does:** Creates a new Git repository in the current directory by creating a `.git` folder.

**Expected output:**
```
Initialized empty Git repository in D:/Documents/SecondLife/NewScripts/.git/
```

## Step 3: Add Remote Repository

```powershell
git remote add origin https://github.com/YourUsername/YourRepo.git
```

**What it does:** Links your local repository to a remote repository on GitHub.
- `origin` is the default name for the remote repository
- Replace `YourUsername` and `YourRepo` with your actual GitHub username and repository name

**Example:**
```powershell
git remote add origin https://github.com/ScottMcGinn/myNotes.git
```

## Step 4: Stage Files for Commit

```powershell
git add .
```

**What it does:** Stages all files in the current directory (and subdirectories) for commit.
- The `.` means "all files in current directory"
- Files listed in `.gitignore` will be excluded automatically

**Alternatives:**
```powershell
# Add a specific file
git add filename.md

# Add multiple specific files
git add file1.md file2.md

# Add all files of a certain type
git add *.md
```

## Step 5: Commit the Staged Files

```powershell
git commit -m "Your commit message here"
```

**What it does:** Saves the staged changes to your local repository with a descriptive message.
- `-m` flag allows you to include the message inline
- Use descriptive messages that explain what changed

**Example:**
```powershell
git commit -m "Add AWS CLI guides and gitignore"
```

**Expected output:**
```
[master (root-commit) f159c40] Add AWS CLI guides and gitignore
 4 files changed, 966 insertions(+)
 create mode 100644 .gitignore
 create mode 100644 AWS_CLI_Guide.md
 create mode 100644 EC2_Instance_Guide.md
```

## Step 6: Push to GitHub

```powershell
git push -u origin master
```

**What it does:** Uploads your committed changes to the remote repository on GitHub.
- `-u` sets the upstream tracking reference (you only need this the first time)
- `origin` is the remote repository name
- `master` is the branch name (some repos use `main` instead)

**Expected output:**
```
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 16 threads
Compressing objects: 100% (6/6), done.
Writing objects: 100% (7/7), 12.10 KiB | 6.05 MiB/s, done.
Total 7 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
To https://github.com/ScottMcGinn/myNotes.git
 * [new branch]      master -> master
branch 'master' set up to track 'origin/master'.
```

## Common Git Commands for Future Use

### Check Status of Your Repository
```powershell
git status
```
Shows which files are modified, staged, or untracked.

### View Commit History
```powershell
git log
```
Shows the history of commits.

### Pull Latest Changes from GitHub
```powershell
git pull
```
Downloads and merges changes from the remote repository.

### Quick Add, Commit, and Push Workflow
```powershell
# Stage all changes
git add .

# Commit with message
git commit -m "Description of changes"

# Push to GitHub
git push
```

Note: After the initial `git push -u origin master`, you can use just `git push` for subsequent pushes.

### View Remote Repository URL
```powershell
git remote -v
```
Shows the URLs of your remote repositories.

### Create and Switch to a New Branch
```powershell
git checkout -b new-branch-name
```

### Switch to an Existing Branch
```powershell
git checkout branch-name
```

## Understanding .gitignore

The `.gitignore` file tells Git which files or patterns to ignore.

**Example `.gitignore`:**
```
# Ignore all LSL files
*.lsl

# Ignore node modules
node_modules/

# Ignore log files
*.log

# Ignore environment variables
.env
```

**What gets ignored:**
- Any file matching the patterns in `.gitignore` will NOT be tracked by Git
- These files won't appear in `git status` or be staged with `git add .`
- Useful for excluding sensitive data, compiled files, or language-specific scripts

## Troubleshooting

### Authentication Required
If GitHub asks for credentials:
- Use a **Personal Access Token** instead of your password
- Generate one at: GitHub Settings → Developer Settings → Personal Access Tokens

### Push Rejected
```powershell
# If the remote has changes you don't have locally
git pull --rebase
git push
```

### Wrong Remote URL
```powershell
# Remove the incorrect remote
git remote remove origin

# Add the correct one
git remote add origin https://github.com/YourUsername/YourRepo.git
```

### Undo Last Commit (Keep Changes)
```powershell
git reset --soft HEAD~1
```

### Undo Last Commit (Discard Changes)
```powershell
git reset --hard HEAD~1
```
⚠️ **Warning:** This permanently deletes your changes!

## Summary of Workflow

1. **Initialize** - `git init`
2. **Connect to GitHub** - `git remote add origin <url>`
3. **Stage files** - `git add .`
4. **Commit** - `git commit -m "message"`
5. **Push** - `git push -u origin master`

For future changes:
1. Make changes to your files
2. `git add .`
3. `git commit -m "describe changes"`
4. `git push`

Your files are now version-controlled and backed up on GitHub!
