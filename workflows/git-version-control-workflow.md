---
description: Git & Version Control Workflow
auto_execution_mode: 1
---

# Git & Version Control Workflow

## Pre-commit Checklist

- [ ] `bun format` run
- [ ] `bun lint` passes
- [ ] Memory updated if needed
- [ ] Todo status updated

## Commit Message Template

```markdown
type(scope): description

- What was changed
- Why it was changed
- Any breaking changes

Closes #issue-number
```

## Branch Naming

- Features: `feature/page-name`
- Fixes: `fix/issue-description`
- Chores: `chore/update-dependencies`
