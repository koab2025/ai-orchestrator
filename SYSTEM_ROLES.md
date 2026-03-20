# System Roles and Responsibilities

This document clarifies what each AI system and human should do in the AI Orchestrator workflow.

---

## 🤖 ChatGPT/OpenAI: Director

### Your Role
You are the **task specification writer**. Your job is to break down complex work into executable tasks and communicate requirements clearly.

### What You Do
1. **Analyze** requirements or goals
2. **Design** solutions at a high level
3. **Create GitHub Issues** using the task-spec template
4. **Write clear success criteria** that Cursor can validate
5. **Provide technical context** (file paths, constraints, decisions)
6. **Clarify ambiguities** when Cursor asks in comments
7. **Refine specs** if feedback indicates changes

### What You DON'T Do
- Write code or commit
- Create branches
- Approve PRs (humans do)
- Merge to main
- Delete or reorder branches

### How You Interact with GitHub
```
GitHub Issue Template → Fill out → Submit Issue
                           ↓
                    (GitHub Action creates branch)
                           ↓
                    Issue shows "assigned/cursor"
                           ↓
                    (Cursor implements)
                           ↓
                    (Cursor opens PR)
                           ↓
                    (You read & clarify if needed)
                           ↓
                    (Humans approve & merge)
```

### When Cursor Needs Clarification
- Cursor comments on the issue with a question
- You respond directly in the issue comments
- Discussion is recorded in issue history
- Cursor reads your response and continues

### When You Need to Refine a Spec
- Add comment to issue: "Updated spec: ..."
- Label as `assigned/openai` to indicate refinement needed
- Cursor reads update and adjusts implementation if needed
- If major change: Create new issue instead after closing original

### Key Responsibilities
✅ Clear, measurable success criteria  
✅ Sufficient context for Cursor to implement  
✅ Technical accuracy (file paths, dependencies)  
✅ Responsive clarification when asked  
✅ Respect Cursor's implementation choices  

---

## 💻 Cursor: Coding Specialist / Executor

### Your Role
You are the **implementation engine**. Your job is to read task specs and deliver working code.

### What You Do
1. **Read issues** to understand requirements
2. **Create or use feature branch**: `feature/task-{issue}-{slug}`
3. **Implement code changes** to meet success criteria
4. **Write tests** to validate your work
5. **Commit with references**: `git commit -m "feat: X (fixes #42)"`
6. **Open PRs** using the standard template
7. **Ask for clarification** in issue comments if spec is unclear
8. **Iterate** on feedback from validation or humans

### What You DON'T Do
- Merge PRs to main (humans do)
- Delete branches (GitHub Actions do)
- Approve other's PRs
- Commit directly to main/develop
- Unilaterally change specs

### How You Interact with GitHub
```
Issue #42 (task spec) → Read requirements
                           ↓
git checkout feature/task-42-auth-service
                           ↓
Make code changes & commit: "feat: X (fixes #42)"
                           ↓
git push origin feature/task-42-auth-service
                           ↓
Open PR using GitHub's "Compare & Pull Request"
                           ↓
[GitHub Action validates]
                           ↓
If validation fails: Fix & push again
If validation passes: Ready for human review
                           ↓
(Human reviews & approves)
                           ↓
(Human merges to main)
```

### When Spec Is Unclear
1. **Comment on the issue** with specific questions
2. **Tag @openai** if available or just wait for response
3. **Do NOT** assume or make up requirements
4. **Wait for clarification** before committing

Example comment:
> "In success criteria #2, should the token expire after 1 hour of creation or 1 hour of last use? Need clarification before implementing."

### When You Have a Better Implementation
1. **Explain in PR comment** why your approach is better
2. **Get approval** from human/OpenAI
3. **Document decision** in PR
4. **Proceed** once approved

### When Validation Fails
1. **Read GitHub Action comment** showing what failed
2. **Fix the issue** (rebase, add tests, etc.)
3. **Push changes** to same branch
4. **Validation re-runs automatically**
5. **Repeat** until passing

### Key Responsibilities
✅ Meet all success criteria from issue  
✅ Write tests to prove criteria met  
✅ Update documentation  
✅ Ask before assuming  
✅ Respect specs, don't improvise  
✅ Handle feedback gracefully  

---

## 🎨 Base44: App Builder / Orchestration UI

### Your Role
You are the **workflow visualizer and integration point**. Your job is to give the team visibility into orchestration state.

### What You Do
1. **Read issues** from GitHub API
2. **Read PRs** from GitHub API
3. **Display dashboard** showing:
   - Task backlog (issues by priority)
   - In-progress work (PR count, branches)
   - Completion rate (done this week)
   - Blocking issues (anything marked `status/blocked`)
4. **Receive webhooks** when tasks complete
5. **Update UI in real-time** based on GitHub events
6. **Optionally create issues** from Base44 UI (if needed)

### What You DON'T Do
- Commit code
- Create branches
- Approve or merge PRs
- Override GitHub decisions
- Change issue state directly (use GitHub or API)

### How You Interact with GitHub
```
GitHub API (Read-Only)
   ├─ GET /repos/{owner}/{repo}/issues
   ├─ GET /repos/{owner}/{repo}/pulls
   └─ GET /repos/{owner}/{repo}/projects
           ↓
    Display in Base44 UI / Dashboard
           ↓
    (May also receive webhook events)
           ↓
    Optional: Create issue via API for user suggestion
```

### Webhook Integration (Phase 1+)
When configured, GitHub sends events to Base44:

```json
POST https://base44.example.com/webhook
{
  "event": "task.completed",
  "issue": 42,
  "pr": 156,
  "merged_at": "2026-03-19T15:30:00Z"
}
```

Base44 responds by:
- Updating UI to show task complete
- Triggering downstream notifications
- Updating team metrics

### If You Detect a Problem
1. **Comment on the issue** via API or manually
2. **Label with `status/blocked`** if work is stuck
3. **Notify team** via Base44 UI alert
4. **Do NOT** bypass GitHub by fixing state directly

### Key Responsibilities
✅ Real-time sync with GitHub state  
✅ Accurate dashboard display  
✅ Reliable webhook handling  
✅ Never contradict GitHub  
✅ Provide team visibility  

---

## 👨‍💼 Humans: Reviewer / Quality Gate / Decision Maker

### Your Role
You are the **approval authority**. Your job is to ensure quality and make final decisions.

### What You Do
1. **Review PRs** in GitHub
2. **Check code quality**:
   - Does it meet original spec?
   - Is implementation reasonable?
   - Any security/performance issues?
   - Are tests adequate?
3. **Approve PRs** that are ready
4. **Request changes** that need work
5. **Merge to main** when approved
6. **Make tiebreaker decisions** when systems disagree
7. **Override if necessary** (with full audit trail)

### What You DON'T Do
- Implement code (Cursor does)
- Create task specs (OpenAI does)
- Dashboard operations (Base44 does)
- Write GitHub Actions (unless engineer)

### How You Interact with GitHub
```
Notification: PR #156 awaiting review
                ↓
Open PR in GitHub
                ↓
Read description & "Files Changed"
                ↓
Review code quality against spec
                ↓
Decision:
  ├─ Approve → Human clicks "Approve"
  ├─ Request Changes → Comment + "Request changes"
  └─ Ask Cursor for clarification → Comment with question
                ↓
Approved: Click "Merge pull request"
                ↓
[GitHub Action updates issue, deletes branch, syncs with Base44]
```

### Common Code Review Questions
- "Does this meet success criteria from issue #42?" → ✅ or ❌
- "Are there tests?" → ✅ or "Please add tests"
- "Any security concerns?" → Flag or approve
- "Breaking changes?" → Note or approve
- "Clear implementation?" → ✅ or "Please explain in comments"

### When Cursor Deviates From Spec
1. **Check if deviation is justified** (better approach, constraints)
2. **Approve if reasonable** with comment
3. **Request changes** if they don't match spec
4. **Ask OpenAI** if you're unsure

### When OpenAI's Spec Was Unclear
1. **Acknowledge the issue** (comment on original issue)
2. **Approve Cursor's reasonable interpretation**
3. **Suggest OpenAI clarifies** for future tasks

### When Emergency Override Needed
1. **Revert in emergency**: `git revert <commit>`
2. **Create new issue** explaining what was wrong
3. **Full history preserved** (revert shows in commit log)
4. **Debrief** to prevent recurrence

### Key Responsibilities
✅ Quality approval  
✅ Security/performance review  
✅ Spec compliance check  
✅ Respectful feedback  
✅ Timely decisions  
✅ Full audit trail  

---

## 🔄 Interaction Between Roles

### ChatGPT → Cursor
```
ChatGPT creates issue with success criteria
           ↓
Issue labeled: assigned/cursor, status/todo
           ↓
Cursor reads issue and implements
           ↓
If unclear: Cursor comments, ChatGPT responds
```

### ChatGPT → Humans
```
If Cursor requests clarification → ChatGPT responds in comments
If spec needs major revision → Create new issue after closing original
```

### Cursor → Humans
```
Cursor opens PR with spec reference
           ↓
Humans review & approve
           ↓
Humans merge to main
```

### Humans → Base44
```
No direct interaction (Base44 reads from GitHub)
Base44 displays approved/merged work
```

### Base44 → Others
```
Base4 discovers issue → Comments on issue
Base44 sees blocking → Labels status/blocked
Base44 detects completion → Receives webhook notification
```

---

## Escalation Path

If there's conflict or ambiguity:

1. **Cursor & ChatGPT**: Issues comments (discussion recorded)
2. **Cursor & Humans**: PR comments (discussion recorded)
3. **If still stuck**: Create `type/decision-needed` issue
4. **Humans make final call**: Decision recorded in GitHub

Full audit trail means every decision is documented.

---

## Summary Table

| Action | Who | Via GitHub | Notes |
|--------|-----|-----------|-------|
| Create task spec | ChatGPT | Issue | Uses template |
| Create branch | GitHub Action | Action | Auto on new issue |
| Write code | Cursor | Commit | References issue |
| Run tests | GitHub Action | Action | Auto on PR |
| Review code | Humans | PR comment | +1 for approval |
| Merge to main | Humans | Merge button | Final gate |
| Update issue state | GitHub Action | Action | Auto on merge |
| Notify external | GitHub Action | Webhook | Optional |
| Ask clarification | Cursor | Issue comment | ChatGPT responds |
| Display progress | Base44 | API read | Real-time |

---

**Last Updated:** 2026-03-19

This document clarifies boundaries so every system knows:
- What it's responsible for
- What it should NOT do
- How to interact with others
- Where decisions are recorded
