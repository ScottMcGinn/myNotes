# GitHub Pull Request Review TL;DR

Quick reference guide for reviewing Pull Requests (PRs) on GitHub.

## What is a Pull Request?

A **Pull Request (PR)** is a request to merge code changes from one branch into another. It's GitHub's way of proposing, discussing, and reviewing code before it becomes part of the main codebase.

## Before You Start

### Quick Checklist
- [ ] Read the PR description and linked issues
- [ ] Check the branch name (should be descriptive)
- [ ] Look at the number of changed files and lines
- [ ] Verify CI/CD checks are passing
- [ ] Check for merge conflicts

## What to Check - The Essentials

### 1. **PR Description & Context**
```
✅ Good PR Description:
- Clear title explaining what changed
- Why the change was made
- What problem it solves
- Link to related issue/ticket
- Testing steps
- Screenshots (for UI changes)

❌ Bad PR Description:
- "Fixed stuff"
- "Updates"
- No context or explanation
```

### 2. **Code Changes Tab**

#### Files Changed
- **How many files?** - Large PRs (>500 lines) are hard to review
- **What types of files?** - Logic, tests, config, docs
- **Deleted files?** - Make sure they're no longer needed

#### Code Quality
```typescript
✅ Look for:
- Clear, readable code
- Proper naming (variables, functions)
- Comments for complex logic
- No hardcoded values (use constants)
- Error handling
- No console.log() or debug code left behind

❌ Red flags:
- Overly complex code
- Copy-pasted code blocks
- Magic numbers (unexplained values)
- Missing error handling
- Commented-out code
- TODO comments without tickets
```

### 3. **Tests**

```
✅ Tests should:
- Cover new functionality
- Test edge cases
- Test error scenarios
- Have descriptive test names
- Actually test what they claim to test

❌ Warning signs:
- No tests for new features
- Tests that always pass
- Tests commented out
- Flaky/random failures
```

### 4. **CI/CD Checks**

```
✅ All checks passing:
- Unit tests ✓
- Integration tests ✓
- Linting ✓
- Build ✓
- Code coverage ✓
- Security scans ✓

❌ Failed checks:
- Don't approve until fixed
- Ask for explanation if unclear
- Verify fixes don't just skip tests
```

### 5. **Security Concerns**

```
🚨 Security red flags:
- Exposed API keys, passwords, tokens
- Hardcoded credentials
- SQL injection vulnerabilities
- XSS vulnerabilities
- Unvalidated user input
- Disabled security features
- Dependencies with known vulnerabilities
- Overly permissive access controls
```

### 6. **Performance**

```
⚡ Check for:
- Unnecessary database queries (N+1 queries)
- Large loops or nested loops
- Memory leaks
- Blocking operations
- Unoptimized images/assets
- Missing pagination on lists
- Inefficient algorithms
```

## What to Look For - By Category

### Code Structure
- [ ] Functions are focused (single responsibility)
- [ ] No overly long functions (>50 lines is suspicious)
- [ ] Proper separation of concerns
- [ ] Consistent with existing code style
- [ ] DRY principle (Don't Repeat Yourself)

### Error Handling
- [ ] Try/catch blocks where needed
- [ ] Meaningful error messages
- [ ] Errors are logged appropriately
- [ ] User-friendly error displays
- [ ] No swallowed errors (empty catch blocks)

### Database Changes
- [ ] Migrations are reversible
- [ ] Indexes added where needed
- [ ] No breaking schema changes
- [ ] Data migrations tested
- [ ] Backup plan exists

### API Changes
- [ ] Backwards compatible (or versioned)
- [ ] Input validation
- [ ] Rate limiting considered
- [ ] Authentication/authorization checked
- [ ] Documentation updated
- [ ] Error responses are meaningful

### Frontend/UI Changes
- [ ] Responsive design (mobile, tablet, desktop)
- [ ] Accessibility (ARIA labels, keyboard navigation)
- [ ] Cross-browser compatibility
- [ ] Loading states
- [ ] Error states
- [ ] Empty states
- [ ] No console errors

### Documentation
- [ ] README updated if needed
- [ ] API docs updated
- [ ] Code comments for complex logic
- [ ] Changelog updated
- [ ] Configuration examples provided

## GitHub PR Interface - What Everything Means

### PR Status Icons
```
✅ Green checkmark - All CI checks passed
❌ Red X - CI checks failed
🟡 Yellow dot - Checks in progress
⚪ Gray circle - Checks pending/not run
🔄 Purple - Review required
✓ Approved - Ready to merge
```

### Review States
```
✅ Approved - You approve the changes
💬 Comment - General feedback, no approval/rejection
❌ Request Changes - Issues must be fixed before merge
```

### Conversation Tab
- Read all comments from other reviewers
- Check if your concerns were already raised
- Look at author's responses to feedback
- See if discussions were resolved

### Commits Tab
- **Number of commits** - Many small commits or few large ones?
- **Commit messages** - Descriptive or vague?
- **Commit history** - Clean or messy?

### Checks Tab
- **CI/CD status** - All passing?
- **Code coverage** - Did it decrease?
- **Linting errors** - Any new issues?
- **Security scans** - Any vulnerabilities?

### Files Changed Tab
- **Diff view** - Unified (single column) or Split (side-by-side)
- **Green lines** - Additions
- **Red lines** - Deletions
- **Collapsed files** - Click to expand large files

## Common Review Comments

### Requesting Changes
```markdown
**Code Quality:**
- Consider extracting this into a separate function for reusability
- This could be simplified using [method/pattern]
- Magic number here - should be a named constant

**Logic Issues:**
- Edge case: What happens if [scenario]?
- This could cause a race condition when [situation]
- Potential null pointer exception here

**Best Practices:**
- Should use async/await instead of callbacks
- Missing error handling for this API call
- This violates the Single Responsibility Principle

**Testing:**
- Add tests for the error scenario
- Missing test for edge case when [condition]
- This test doesn't actually verify the behavior
```

### Asking Questions
```markdown
- Why was this approach chosen over [alternative]?
- Is this backwards compatible with [version]?
- Can you explain what this code block does?
- Is there a ticket for this TODO comment?
- Should this be configurable instead of hardcoded?
```

### Giving Praise
```markdown
✅ Nice refactoring!
✅ Great test coverage!
✅ Love the descriptive variable names
✅ Excellent error handling
✅ Good catch on that edge case
```

## Merge Strategies - Which One?

### Create a Merge Commit
- Preserves all commits from feature branch
- Shows complete history
- Can make history messy with many small commits

### Squash and Merge
- Combines all commits into one
- Cleaner main branch history
- Loses individual commit details

### Rebase and Merge
- Adds commits individually without merge commit
- Linear history
- Can cause issues if not done carefully

**Most common:** Squash and merge for cleaner history

## When to Approve

### ✅ Approve When:
- All CI checks pass
- Code meets quality standards
- Tests are adequate
- No security concerns
- Documentation is updated
- All discussions resolved
- You understand the changes
- You'd be comfortable maintaining this code

### ❌ Request Changes When:
- CI checks failing
- Security vulnerabilities
- Missing tests
- Code quality issues
- Breaking changes without migration plan
- Unresolved concerns from other reviewers
- You don't understand what the code does

### 💬 Comment (No Approval) When:
- Nitpicks that aren't blockers
- Suggestions for future improvements
- Questions for your own understanding
- General discussion points

## Red Flags 🚩

```
🚩 STOP - Do Not Merge:
- Secrets/credentials in code
- Disabled security features
- "Testing in production" mindset
- Breaking changes without notice
- No tests at all
- All tests skipped/commented out
- CI bypassed or forced
- Merge conflicts not resolved
- "Works on my machine" with no testing
- Large PR with "refactoring" and new features mixed
```

## Pro Tips

1. **Start with small stuff** - Check formatting, typos first
2. **Run the code locally** - Don't just read it
3. **Test the changes** - Follow the test steps in the PR description
4. **Review incrementally** - For large PRs, review in multiple sessions
5. **Use GitHub suggestions** - Suggest code changes directly
6. **Be respectful** - Critique the code, not the person
7. **Be specific** - "This is confusing" vs "Consider renaming X to Y"
8. **Explain the why** - Don't just say "change this", explain why
9. **Use the GitHub CLI** - `gh pr checkout 123` to test locally
10. **Check the diff carefully** - Unintended changes often sneak in

## Quick GitHub PR Review Workflow

```powershell
# 1. Checkout the PR locally
gh pr checkout 123

# 2. Install dependencies (if needed)
npm install

# 3. Run tests
npm test

# 4. Run the app
npm start

# 5. Test the changes manually

# 6. Go back to your branch
git checkout main

# 7. Leave review on GitHub
```

## Time-Saving Shortcuts

### GitHub Web UI
- `?` - Show keyboard shortcuts
- `e` - Edit file
- `t` - File finder
- `l` - Jump to line
- `w` - Switch branch/tag
- `y` - Permalink to file

### Review Shortcuts
- `.` - Open in github.dev (web editor)
- `Cmd/Ctrl + K` → `PR` - Quick search PRs
- `Cmd/Ctrl + /` - Toggle comment preview

## Review Size Guidelines

```
📊 PR Size Recommendations:

Tiny (1-50 lines)
⏱️ Time: 5-10 min
👍 Easy to review thoroughly

Small (50-200 lines)
⏱️ Time: 10-30 min
👍 Ideal size for review

Medium (200-500 lines)
⏱️ Time: 30-60 min
⚠️ Consider splitting if possible

Large (500-1000 lines)
⏱️ Time: 1-2 hours
⚠️ Should be split into smaller PRs

Huge (1000+ lines)
⏱️ Time: 2+ hours
🚫 Definitely split this up!
```

## Sample Review Checklist

```markdown
### Code Review Checklist

**Functionality**
- [ ] Code does what PR describes
- [ ] Edge cases handled
- [ ] Error scenarios covered
- [ ] No obvious bugs

**Code Quality**
- [ ] Clear and readable
- [ ] Follows project conventions
- [ ] No code duplication
- [ ] Proper abstractions

**Tests**
- [ ] Tests exist and pass
- [ ] Good test coverage
- [ ] Tests are meaningful
- [ ] Edge cases tested

**Security**
- [ ] No exposed secrets
- [ ] Input validation
- [ ] Authentication/authorization
- [ ] No SQL injection risks

**Performance**
- [ ] No obvious bottlenecks
- [ ] Efficient algorithms
- [ ] Database queries optimized
- [ ] No memory leaks

**Documentation**
- [ ] Code comments where needed
- [ ] README updated
- [ ] API docs updated
- [ ] Migration guide (if breaking)

**CI/CD**
- [ ] All checks passing
- [ ] No test failures
- [ ] Build succeeds
- [ ] Coverage maintained

**Final Check**
- [ ] Merge conflicts resolved
- [ ] Branch up to date with main
- [ ] Ready to deploy
```

## Common Mistakes to Avoid

### As a Reviewer
❌ Approving without actually reviewing
❌ Being overly nitpicky about style (use linters)
❌ Blocking on personal preferences
❌ Not explaining your reasoning
❌ Leaving vague comments
❌ Not testing the changes locally

### As a PR Author
❌ Creating huge PRs
❌ Mixing refactoring with new features
❌ Not responding to comments
❌ Merging with unresolved comments
❌ Skipping tests to pass CI
❌ Poor PR description

## Useful GitHub Features

### Inline Comments
```
Click on line number → Add comment
Shift+click to select range → Add multi-line comment
```

### Suggested Changes
```markdown
```suggestion
const userName = user.name;
` ``
```

### Review Requests
- Request specific reviewers
- Use CODEOWNERS file for auto-assignment
- Re-request review after changes

### Draft PRs
- Mark PR as draft while in progress
- CI runs but can't merge
- Convert to ready for review when done

## Resources

- [GitHub PR Documentation](https://docs.github.com/en/pull-requests)
- [GitHub Code Review Guidelines](https://github.com/features/code-review)
- [Google Engineering Practices - Code Review](https://google.github.io/eng-practices/review/)
- [The Art of Code Review (Talk)](https://www.youtube.com/watch?v=6tQKUruKU_g)

## Quick Decision Tree

```
Is PR ready to review?
├─ No → Ask author to mark as ready / resolve conflicts
└─ Yes → Continue

Are CI checks passing?
├─ No → Request fixes
└─ Yes → Continue

Do you understand the changes?
├─ No → Ask questions, request documentation
└─ Yes → Continue

Are there security concerns?
├─ Yes → Request changes immediately
└─ No → Continue

Are tests adequate?
├─ No → Request additional tests
└─ Yes → Continue

Is code quality acceptable?
├─ No → Request changes with specific feedback
└─ Yes → Continue

Would you be comfortable maintaining this?
├─ No → Identify and communicate concerns
└─ Yes → APPROVE ✅
```

Remember: **It's better to ask too many questions than to approve code you don't understand!**
