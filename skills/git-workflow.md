---
name: git-workflow
description: Manage git operations (commits, branches, PRs)
triggers:
  - "commit"
  - "push"
  - "branch"
  - "pull"
  - "merge"
  - "stash"
  - "git"
---

# Git Workflow Skill

Manages git operations following project conventions.

## Supported Operations

### Commit Workflow

1. Check git status for changes
2. Stage relevant files
3. Generate commit message following conventional commits
4. Commit with appropriate scope

### Commit Message Convention

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting
- `refactor`: Code refactoring
- `test`: Tests
- `chore`: Maintenance

**Examples**:
```
feat(auth): add login endpoint
fix(api): resolve response timeout
docs(readme): update installation guide
```

## Behavior

1. **Analyze changes**: Use `git status` and `git diff`
2. **Suggest scope**: Based on changed files
3. **Generate message**: Following conventional commits
4. **Confirm before commit**: Show full message, ask for approval
5. **Execute**: Run `git commit`

## Branch Naming Convention

```
<type>/<ticket-id>-<short-description>

Examples:
- feature/PROJ-123-user-auth
- fix/PROJ-456-login-error
- chore/update-dependencies
```

## Output Format

```
=== Git Workflow ===

Status: 3 files changed
- src/auth/login.ts (modified)
- src/auth/logout.ts (new)
- tests/auth.test.ts (modified)

Suggested commit:
feat(auth): add login and logout endpoints

Stage files? [y/N]: y
Commit? [y/N]: y

✓ Committed: feat(auth): add login and logout endpoints
```

## Operations

| Operation | Command | Description |
|-----------|---------|-------------|
| `status` | `git status` | Show working tree status |
| `commit` | `git commit -m "message"` | Commit staged changes |
| `push` | `git push` | Push to remote |
| `branch` | `git checkout -b` | Create/switch branch |
| `stash` | `git stash` | Stash changes temporarily |

## Safety Checks

- **No force push** on main/master
- **Confirm** before commit
- **Warn** if committing secrets
- **Require** branch name convention

## Notes

- Always use `--no-verify` with caution
- Check for `.gitignore` violations
- Warn if large files (>5MB) being committed
- Suggest `git add -p` for partial staging