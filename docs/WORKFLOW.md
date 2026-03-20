# Day-to-Day Operations: AI Orchestrator Workflow

## For ChatGPT/OpenAI: Creating Task Specs

### Step 1: Understand the Work
- Analyze what needs to be done
- Break down complex work into discrete tasks
- Identify success criteria and constraints

### Step 2: Create Issue Using Task-Spec Template
1. Go to GitHub → New Issue
2. Select template: **Task Spec (ChatGPT)**
3. Fill out:
   - **Title**: `[TASK] Brief one-line description`
   - **Executive Summary**: One sentence of desired end state
   - **Success Criteria**: 3-5 measurable checkboxes
   - **What to Change**: Pseudo-diff or scope clarification
   - **Context for Cursor**: Technical details, file paths, constraints
   - **Metadata**: Component, Effort, Type, Priority

### Step 3: Submit
- Click "Submit new issue"
- GitHub automatically applies labels
- GitHub Action creates feature branch
- Issue is now **ready for Cursor**

### Template Example
```markdown
[TASK] Implement user authentication service

Executive Summary:
Create a JWT-based authentication service that issues 1-hour tokens and validates them on protected endpoints.

Success Criteria:
- [ ] POST /auth/token returns JWT token
- [ ] Token expires after 1 hour
- [ ] Expired tokens rejected with 401
- [ ] Tests confirm all above
- [ ] README.md updated with auth docs

What to Change:
- New file: src/auth/jwt-service.py
- Modify: src/api/middleware.py (add auth check)
- New tests: tests/auth_test.py

Context for Cursor:
- Use PyJWT library (already in requirements.txt)
- Token secret in environment variable: AUTH_SECRET
- Middleware should check Authorization: Bearer {token} header
- Return 401 if missing or invalid

Metadata:
- Component: api-layer
- Effort: medium
- Type: feature
- Priority: high
```

### Refinement
If issue becomes unclear or changes needed:
- **Comment in the issue** with clarifications
- Use `@cursor-bot` to notify Cursor if needed
- Label with `assigned/openai` if spec needs revision
- Cursor will ask in comments before starting if confused

---

## For Cursor: Implementing Tasks

### Step 1: Get Task
1. GitHub notification: New issue assigned, labeled `assigned/cursor`
2. Read the issue for complete requirements
3. Check feature branch exists: `feature/task-{issue}-{slug}`

### Step 2: Implement
```bash
# Clone (if not already)
git clone <repo>

# Checkout feature branch
git checkout feature/task-42-auth-service

# Make your changes
# (edit files, add tests, update docs)

# Commit with reference to issue
git commit -m "feat: implement JWT auth service (fixes #42)"
git commit -m "test: add auth validation tests (fixes #42)"
git commit -m "docs: update README with auth flow (fixes #42)"

# Push
git push origin feature/task-42-auth-service
```

### Step 3: Open Pull Request
1. GitHub shows "Compare & pull request" button
2. Click it
3. Template auto-fills
4. Fill in:
   - **What Changed**: Summary of your implementation
   - **How to Test**: Exact steps to verify it works
   - **Implementation Notes**: Any deviations from spec
   - Verify all checkboxes match original issue criteria

### Step 4: Address Validation
- GitHub Actions run automatically
- Check for ✅ or ❌ results
- If ❌:
  - Fix issues (rebase, add tests, etc.)
  - Push again
  - Workflow re-runs automatically
- If ✅: You're ready for human review

### Step 5: Wait for Approval
- Human reviewer examines your PR
- Approves or requests changes
- When approved: **DO NOT MERGE**
- Human merges to complete task

### Checklist Before Opening PR
- [ ] All success criteria from issue met
- [ ] Tests pass locally: `npm test` or `pytest`
- [ ] No breaking changes (or approved)
- [ ] Documentation updated
- [ ] PR links to issue with `Closes #XXX`

### If Issue Becomes Unclear
- **Comment on the issue** with specific questions
- Tag `@yourusername` (Cursor bot if it's an AI)
- Wait for OpenAI response in comments
- Do NOT assume or make up requirements

---

## For Humans: Review & Merge

### Step 1: Monitor
- Check GitHub Project Board: "In Review" column
- Or watch email notifications
- Or check daily in Base44 dashboard

### Step 2: Code Review
1. Open the PR (GitHub shows link)
2. Click "Files Changed" tab
3. Review Cursor's code:
   - Does it meet original spec?
   - Is the implementation reasonable?
   - Any security/performance concerns?
   - Are tests adequate?

### Step 3: Decision
**If approved:**
1. Click "Approve" button in GitHub
2. Click "Merge pull request"
3. Select "Squash and merge" or "Create a merge commit" (team choice)
4. Confirm
5. GitHub Actions automatically:
   - Updates original issue: `status/done`
   - Deletes feature branch
   - Notifies Base44 webhook (if configured)
   - Closes the issue

**If changes needed:**
1. Click "Request changes"
2. Comment with specific feedback
3. Cursor receives notification and iterates

### Step 4: Verify Completion
- Issue moved to "Done" column in project board
- Issue marked `status/done`
- Base44 dashboard reflects completion
- Code merged to main and deployed

---

## For Base44: Orchestration Dashboard

### What Base44 Reads
- Issue list (with labels, status, priority)
- PR list (with links, status, validation results)
- Project board (Backlog → Ready → In Progress → Review → Done)
- Commit history (with linked issues)
- Webhook notifications (on task completion)

### What Base44 Displays
- Task backlog and priority
- Current progress (how many in progress, in review, etc.)
- Team velocity (tasks completed per week)
- Blocking issues (tasks waiting on others)
- System status (all issues linked to PRs, passing validation, etc.)

### Base44 Does NOT Write
- Code (only Cursor writes code)
- PRs (only Cursor creates PRs)
- Issue metadata (only GitHub Actions update)
- Branches (only GitHub Actions create/delete)

### Webhooks
If configured, Base44 receives:
```json
{
  "event": "task.completed",
  "issue": 42,
  "pr": 156,
  "merged_at": "2026-03-19T15:30:00Z"
}
```

This triggers Base44 to:
- Update UI to show task complete
- Ensure dashboard stays in sync with reality
- Optionally trigger downstream systems

---

## Common Scenarios

### Scenario 1: Unclear Spec
1. **Cursor discovers** spec is ambiguous while implementing
2. **Cursor comments** on original issue with specific questions
3. **OpenAI responds** in those comments
4. **Cursor reads comments** and continues implementation
5. **Issue stays open** (GitHub labels stays `status/in-progress`)

### Scenario 2: Blocked Work
1. **Cursor discovers** task depends on another task not yet done
2. **Cursor comments** on issue: "Blocked by #99, waiting for..."
3. **Label issue** with `status/blocked`
4. **OpenAI reviews** and prioritizes #99 if needed
5. **Once #99 merges**, Cursor resumes

### Scenario 3: Implementation Differs From Spec
1. **Cursor discovers** spec requires X, but better approach is Y
2. **Cursor explains in PR comments** why Y is better
3. **Human (or OpenAI) approves deviation**
4. **PR merges** with documented decision
5. **History preserved** in commit

### Scenario 4: Test Failures in Validation
1. **Cursor opens PR**
2. **cursor-validation.yml runs tests**
3. **Tests fail**, GitHub Action posts ❌ comment
4. **Cursor fixes code** and pushes again
5. **Tests re-run automatically** (GitHub re-triggers workflow)
6. **Once passing**, ✅ is posted

### Scenario 5: Emergency Override
1. **Human discovers** something critical was missed
2. **Human reverts merge**: `git revert <commit>`
3. **Creates new issue** explaining what was wrong
4. **Original issue reopens** with `status/todo`
5. **Full history preserved** (revert is a commit)

---

## Status Labels and What They Mean

| Label | Meaning | Who Can Change |
|-------|---------|---|
| `status/todo` | Ready for Cursor to start | GitHub Action (auto) or OpenAI (manual) |
| `status/in-progress` | Cursor is actively working | Cursor via comments (optional, auto-updated) |
| `status/review` | PR is open, waiting for approval | GitHub Action (auto on PR open) |
| `status/blocked` | Waiting on external input | Anyone via label |
| `status/done` | Merged and complete | GitHub Action (auto on merge) |

---

## Tools/Access Required

### ChatGPT/OpenAI
- GitHub login
- Permission to create issues
- Access to issue template

### Cursor
- GitHub login
- Clone repository rights
- Permission to push to `feature/task-*` branches
- Permission to open PRs
- NOT permission to merge to main

### Base44
- GitHub API token (read-only)
- Optional: Webhook endpoint to receive task completion events
- Displays projects/issues/PRs from GitHub

### Humans
- GitHub login
- Permission to review PRs
- Permission to approve and merge PRs
- Optional: GitHub.com web access

---

## Troubleshooting

### Issue: PR won't merge (validation fails)
**Solution:** GitHub Action shows ❌ with reason. PR template incomplete? Tests fail? Fix issues and push again.

### Issue: Feature branch doesn't exist
**Solution:** Run: `git checkout -b feature/task-NNN-description`

### Issue: PR shows conflicts
**Solution:** Rebase: `git fetch origin main && git rebase origin/main` then force-push

### Issue: OpenAI needs to clarify spec
**Solution:** Comment on issue, wait for response in comments, don't start implementation until clear

### Issue: Base44 out of sync with GitHub
**Solution:** Refresh Base44 (reads via API). If webhook misconfigured, GitHub is still the source of truth.

---

## Quick Reference Cheat Sheet

| Role | Action | Time |
|------|--------|------|
| OpenAI | Write issue spec | 15-30 min |
| GitHub | Create branch & label | <1 min (auto) |
| Cursor | Implement & test | 1-8 hours |
| Cursor | Open PR | 5 min |
| GitHub | Validate PR | <1 min (auto) |
| Human | Review PR | 15-30 min |
| Human | Merge PR | 1 min |
| GitHub | Update issue & cleanup | <1 min (auto) |
| Base44 | Display in dashboard | Real-time |

---

Last Updated: 2026-03-19
