---
name: git-operations-manager
description: Safe git operations with built-in validations and best practices. Prevents dangerous operations, enforces commit message standards, and provides safe workflows for commits, branches, and pushes.
version: 1.0.0
category: core
priority: high
---

# Git Operations Manager

A core skill that provides safe, validated git operations for all agents. Prevents common git mistakes and enforces best practices.

## Purpose

**Problem:** Git operations are risky and error-prone without proper safeguards
**Solution:** Validated git workflows with safety checks built-in
**Benefit:** Zero git disasters + consistent commit practices

## Capabilities

- ✅ Safe commit workflow with validation
- ✅ Branch creation with naming conventions
- ✅ Status checking with parsed output
- ✅ History analysis for files/modules
- ✅ Commit message generation (conventional commits)
- ✅ Pre-commit safety checks
- ✅ Secret detection
- ✅ Authorship validation before amend

## Safety Principles

### NEVER
- ❌ Force push to main/master
- ❌ Amend commits by other developers
- ❌ Commit files containing secrets (.env, credentials, keys)
- ❌ Skip hooks (--no-verify, --no-gpg-sign)
- ❌ Use hard reset without explicit user request
- ❌ Interactive commands (git rebase -i, git add -i)

### ALWAYS
- ✅ Check authorship before amending
- ✅ Use heredoc for commit messages
- ✅ Validate files before commit
- ✅ Check git status after commits
- ✅ Use conventional commit format
- ✅ Track commits in session reports

## Usage

### Action: status

Gets current git status with parsed output.

**Input:**
```
Skill: git-operations-manager
Action: status
```

**Output:**
- branch: string
- untracked_files: array
- modified_files: array
- staged_files: array
- ahead_behind: object

**Example:**
```
Skill: git-operations-manager
Action: status
```

Returns:
```
Branch: claude/feature-auth-013K9xb2eNbLKYph7MxsxHyg
Untracked Files:
  - src/auth/login.js
  - src/auth/register.js
Modified Files:
  - README.md
Staged Files:
  - src/auth/middleware.js
Status: Clean (ready to commit)
Ahead: 2 commits
Behind: 0 commits
```

### Action: commit

Creates a safe commit with validation and conventional message format.

**Input:**
```
Skill: git-operations-manager
Action: commit
Files: <array of file paths>
Message Type: <feat | fix | docs | refactor | test | chore>
Description: <short description>
Body: <detailed description> (optional)
Session ID: <DDMM_YYYY_NN> (optional, adds to commit message)
```

**Output:**
- commit_sha: string
- commit_message: string
- files_committed: number
- safety_checks_passed: boolean

**Example:**
```
Skill: git-operations-manager
Action: commit
Files:
  - src/auth/login.js
  - src/auth/register.js
  - src/auth/middleware.js
Message Type: feat
Description: Add JWT authentication
Body: |
  Implemented login and registration endpoints with JWT token generation.
  Added middleware for token validation.
Session ID: 1611_2025_01
```

Returns:
```
Commit SHA: a1b2c3d4e5f6
Commit Message:
  feat: Add JWT authentication

  Implemented login and registration endpoints with JWT token generation.
  Added middleware for token validation.

  Session: 1611_2025_01

Files Committed: 3
  ✓ src/auth/login.js
  ✓ src/auth/register.js
  ✓ src/auth/middleware.js

Safety Checks Passed:
  ✓ No secrets detected
  ✓ All files exist
  ✓ No binary files
  ✓ Commit message follows conventional format
```

### Action: branch

Creates a new branch with proper naming convention.

**Input:**
```
Skill: git-operations-manager
Action: branch
Branch Name: <descriptive-name>
Prefix: <feature | bugfix | hotfix | experiment> (optional)
Base: <base-branch-name> (optional, defaults to current)
```

**Output:**
- branch_name: string
- created: boolean
- checked_out: boolean

**Example:**
```
Skill: git-operations-manager
Action: branch
Branch Name: add-user-roles
Prefix: feature
```

Returns:
```
Branch Created: feature/add-user-roles
Checked Out: true
Base: main
Ready to work
```

### Action: history

Analyzes git history for a file or module.

**Input:**
```
Skill: git-operations-manager
Action: history
Path: <file or directory path>
Limit: <number of commits> (optional, default 10)
```

**Output:**
- commits: array of commit objects
- total_commits: number
- contributors: array

**Example:**
```
Skill: git-operations-manager
Action: history
Path: src/auth/
Limit: 5
```

Returns:
```
Commits (5 most recent):
  1. a1b2c3d - feat: Add JWT authentication (2025-11-16, quick-developer)
  2. b2c3d4e - refactor: Extract auth logic (2025-11-15, quality-developer)
  3. c3d4e5f - fix: Handle null user (2025-11-14, quick-developer)
  4. d4e5f6g - test: Add auth tests (2025-11-14, test-developer)
  5. e5f6g7h - feat: Initial auth setup (2025-11-13, code-architect)

Total Commits: 47
Contributors: [quick-developer, quality-developer, test-developer, code-architect]
```

### Action: diff

Shows diff for specific files or entire repo.

**Input:**
```
Skill: git-operations-manager
Action: diff
Files: <array of file paths> (optional, defaults to all)
Staged: <boolean> (optional, shows staged vs unstaged)
```

**Output:**
- diff_output: string
- files_changed: number
- lines_added: number
- lines_removed: number

**Example:**
```
Skill: git-operations-manager
Action: diff
Files:
  - src/auth/login.js
```

Returns:
```
Diff for src/auth/login.js:
+++ b/src/auth/login.js
@@ -10,6 +10,12 @@ async function login(email, password) {
+  // Validate input
+  if (!email || !password) {
+    throw new ValidationError('Email and password required');
+  }
+

Files Changed: 1
Lines Added: 6
Lines Removed: 0
```

### Action: validate-commit

Validates files before committing (secret detection, etc.).

**Input:**
```
Skill: git-operations-manager
Action: validate-commit
Files: <array of file paths>
```

**Output:**
- safe_to_commit: boolean
- issues: array
- warnings: array

**Example:**
```
Skill: git-operations-manager
Action: validate-commit
Files:
  - .env
  - src/auth/login.js
```

Returns:
```
Safe to Commit: false

Issues:
  - .env contains potential secrets (API_KEY=...)
  - .env should be in .gitignore

Warnings:
  - Consider reviewing sensitive data in these files

Recommendation: DO NOT COMMIT .env file
```

## Conventional Commit Format

All commits MUST follow this format:

```
<type>: <short description>

<optional body>

<optional footer>
```

### Types
- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **refactor**: Code refactoring
- **test**: Adding or updating tests
- **chore**: Maintenance tasks
- **style**: Code style changes (formatting)
- **perf**: Performance improvements
- **ci**: CI/CD changes

### Examples

**Feature:**
```
feat: Add user authentication

Implemented JWT-based authentication with login and registration
endpoints. Added middleware for token validation.

Session: 1611_2025_01
```

**Bug Fix:**
```
fix: Handle null reference in user service

Added null check before accessing user.email property to prevent
NullReferenceException.

Fixes #123
Session: 1611_2025_02
```

**Documentation:**
```
docs: Update API documentation

Added examples for authentication endpoints and updated error
response formats.
```

**Refactoring:**
```
refactor: Extract database logic to repository pattern

Moved all database queries from service layer to repository classes
for better separation of concerns.

Session: 1611_2025_01
```

## Safety Checks

### Pre-Commit Checks

**1. Secret Detection**
Scans files for:
- API keys (pattern: `API_KEY=`, `APIKEY:`, etc.)
- Passwords (pattern: `PASSWORD=`, `PWD:`, etc.)
- Tokens (pattern: `TOKEN=`, `JWT:`, etc.)
- Private keys (pattern: `PRIVATE_KEY`, `-----BEGIN`)
- Connection strings (pattern: `mongodb://`, `postgres://`, etc.)

**2. File Validation**
Checks:
- All files exist
- No binary files (unless explicitly allowed)
- Files are readable
- No empty commits

**3. .gitignore Check**
Warns if committing files that should be ignored:
- `.env`, `.env.local`, `.env.*`
- `credentials.json`, `secrets.json`
- `*.key`, `*.pem`
- `node_modules/` (if accidentally staged)

### Pre-Push Checks

**1. Branch Protection**
```
❌ BLOCKED: git push --force origin main
✅ ALLOWED: git push origin feature/add-auth
```

**2. Commit Authorship**
Before amending, checks:
```
git log -1 --format='%an %ae'
```
If author != current user → WARN and ask confirmation

**3. Unpushed Work**
Verifies all local commits are being pushed:
```
git log origin/branch..HEAD
```

## Workflow Patterns

### Pattern 1: Create Feature and Commit

```
# 1. Create branch
Skill: git-operations-manager
Action: branch
Branch Name: add-authentication
Prefix: feature

# 2. [Do your work...]

# 3. Check status
Skill: git-operations-manager
Action: status

# 4. Validate before commit
Skill: git-operations-manager
Action: validate-commit
Files: [src/auth/login.js, src/auth/register.js]

# 5. Commit
Skill: git-operations-manager
Action: commit
Files: [src/auth/login.js, src/auth/register.js]
Message Type: feat
Description: Add user authentication
Session ID: 1611_2025_01
```

### Pattern 2: Bug Fix Workflow

```
# 1. Create bugfix branch
Skill: git-operations-manager
Action: branch
Branch Name: fix-null-reference
Prefix: bugfix

# 2. [Fix the bug...]

# 3. Get file history to understand changes
Skill: git-operations-manager
Action: history
Path: src/services/UserService.cs
Limit: 10

# 4. Commit fix
Skill: git-operations-manager
Action: commit
Files: [src/services/UserService.cs]
Message Type: fix
Description: Handle null reference in getUserById
Body: Added null check before accessing user properties
Session ID: 1611_2025_02
```

### Pattern 3: Check What Changed

```
# 1. Get status
Skill: git-operations-manager
Action: status

# 2. See detailed diff
Skill: git-operations-manager
Action: diff

# 3. Review specific file
Skill: git-operations-manager
Action: diff
Files: [src/auth/login.js]
```

## Integration with Other Skills

### Works With:
- **session-manager**: Include session_id in commit messages
- **json-report-manager**: Track commits in session reports
- **agent-signature-generator**: Files already signed before commit

### Workflow:
```
1. session-manager: Create session → get session_id
2. [Do work and create files]
3. agent-signature-generator: Sign files
4. git-operations-manager: Validate and commit
5. json-report-manager: Record commit in report
```

## Error Prevention

### Common Git Mistakes Prevented

**Mistake 1: Committing Secrets**
```
❌ Without Skill: Commits .env with API keys
✅ With Skill: Detects and blocks commit
```

**Mistake 2: Force Push to Main**
```
❌ Without Skill: git push --force origin main (destroys history)
✅ With Skill: Blocked with error message
```

**Mistake 3: Amending Other's Commits**
```
❌ Without Skill: git commit --amend (overwrites teammate's work)
✅ With Skill: Checks authorship, warns if different
```

**Mistake 4: Bad Commit Messages**
```
❌ Without Skill: "fixed stuff"
✅ With Skill: Enforces "fix: Handle null reference in user service"
```

## Best Practices

1. **ALWAYS validate** before committing
2. **Use conventional commits** for all messages
3. **Include session ID** in commit messages
4. **Check status** after operations
5. **Review diff** before committing
6. **Small, focused commits** over large dumps
7. **Meaningful descriptions** in commit messages

## Configuration

### Commit Message Template

Default template includes:
```
{type}: {description}

{body}

Session: {session_id}
Agent: {agent_name}
```

### Secret Patterns

Default patterns (can be extended):
```regex
API_KEY=.*
PASSWORD=.*
TOKEN=.*
PRIVATE_KEY
mongodb://.*
postgres://.*:.*@
AWS_.*=
STRIPE_.*=
```

## Troubleshooting

### "Secrets detected, commit blocked"
```
Problem: File contains potential secrets
Solution: Remove secrets, use environment variables
```

### "Cannot push to protected branch"
```
Problem: Trying to push to main/master
Solution: Create feature branch instead
```

### "Commit author mismatch"
```
Problem: Trying to amend commit by different author
Solution: Create new commit instead of amending
```

## Advanced Features

### Amend Last Commit (with safety)

```
Skill: git-operations-manager
Action: amend
Files: [new-file.js]
```

This will:
1. Check last commit author
2. Check if commit is pushed
3. If safe, amend commit
4. If not, suggest creating new commit

### Interactive Add (safe version)

```
Skill: git-operations-manager
Action: stage
Files: [file1.js, file2.js]
Interactive: false
```

Stages files without interactive mode (which doesn't work in headless).

## Version History

- **1.0.0** (2025-11-16): Initial implementation with safety checks and conventional commits
