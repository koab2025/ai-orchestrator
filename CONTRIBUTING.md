# Contributing to AI Orchestrator

Welcome to AI Orchestrator! This document explains how to participate in our GitHub-based development workflow.

## Quick Start

Before contributing, please read:
1. [SYSTEM_ROLES.md](SYSTEM_ROLES.md) — Understand your role and boundaries
2. [docs/WORKFLOW.md](docs/WORKFLOW.md) — Learn the task lifecycle
3. [docs/ORCHESTRATION.md](docs/ORCHESTRATION.md) — Understand the architecture

## The MVP Workflow

### If You're the OpenAI/ChatGPT Director
1. Analyze work that needs to be done
2. Break it into discrete tasks
3. Create a GitHub Issue using the **Task Spec** template
4. Include clear success criteria and context
5. Respond to Cursor's clarifying questions in issue comments
6. Review PRs and provide feedback

### If You're a Cursor/Coding Specialist
1. Watch for issues labeled `assigned/cursor` with `status/todo`
2. Read the task spec completely in the Issue
3. Ask clarifying questions in comments if needed
4. `git checkout feature/task-{id}-{slug}`
5. Implement code changes
6. Commit with message: `git commit -m "feat: X (fixes #{issue})"`
7. Open a PR using the template
8. Address validation feedback if needed
9. Don't merge — wait for human approval

### If You're a Human Reviewer
1. Monitor PRs in the "In Review" state
2. Review code against the original task spec
3. Check for security/performance concerns
4. Approve or request changes
5. Once approved, merge to main
6. GitHub Actions will handle cleanup and notifications

### If You're Base44 / Orchestration UI
1. Read GitHub Issues and PRs via the API
2. Display the current state in your dashboard
3. Receive webhook notifications on task completion
4. Never commit code or merge PRs
5. Always respect GitHub as source of truth

## Important Conventions

### Branching
```
feature/task-{issue-id}-{slug}

Example: feature/task-42-auth-service
```
- Always create a branch per issue
- Never work directly on main/develop
- Branch is auto-created when task is marked `assigned/cursor`

### Commit Messages
```
feat: brief description (fixes #{issue})
```
Include the issue reference so commits are linked.

### Pull Requests
- Use the PR template (auto-filled)
- Link to the issue with `Closes #XXX`
- List what changed and how to test
- Check the success criteria from the original issue
- Don't merge yourself — wait for approval

### Labels
| What | Label |
|------|-------|
| Task ready to implement | `status/todo` `assigned/cursor` |
| Someone is working on it | `status/in-progress` |
| PR waiting for review | `status/review` |
| Task is complete | `status/done` |

## Handling Problems

### Issue Spec Is Unclear
- **Don't** assume what you need to do
- **Do** comment on the issue with specific questions
- **Do** wait for OpenAI's response before coding

### Test Validation Fails
- GitHub Action comments on your PR with the error
- Fix the issue locally
- Push again — tests re-run automatically

### PR Has Merge Conflicts
- Rebase: `git fetch origin main && git rebase origin/main`
- Resolve conflicts locally
- Force push: `git push --force-with-lease`

### Code Review Asks for Changes
- Make requested changes
- Commit and push to same branch
- GitHub Auto-updates the PR
- Request re-review when ready

## Questions?

- Check [docs/WORKFLOW.md](docs/WORKFLOW.md) for step-by-step guides
- Check [SYSTEM_ROLES.md](SYSTEM_ROLES.md) for role clarifications
- Comment on relevant GitHub Issue with your question
- Tag @openai or @base44 if seeking specific input

## Code of Conduct

All participants agree to:
- Respect task spec boundaries (don't improvise features beyond spec)
- Respect role boundaries (don't merge your own PRs, don't commit as another role)
- Be responsive (answer clarifying questions quickly)
- Keep GitHub as single source of truth (no ad-hoc decisions)

---

**Happy orchestrating! 🎼**
