# AI Orchestrator - GitHub Integration Architecture

## Overview

This document describes how **GitHub** serves as the operational backbone for coordinating 4 AI/human systems:

1. **OpenAI/ChatGPT** — Director (creates task specs)
2. **Cursor** — Coding specialist (executes tasks)
3. **Base44** — App builder/orchestration UI
4. **GitHub** — Source of truth and control layer

---

## Core Principles

- **GitHub is single source of truth** for tasks, execution state, and decisions
- **Issues = task specs** (ChatGPT output, structured with YAML metadata)
- **Pull requests = executable deliverables** (Cursor's work for review)
- **Branches = isolated task execution contexts** (one per issue)
- **Labels/metadata = routing and context** (indicate who should act next)
- **Commits = atomic work units** with linked issues for full history

---

## Data Model: The Execution Ledger

Every decision, task, and output flows through GitHub:

| Component | Role | Example |
|-----------|------|---------|
| **Issue** | Task specification | Chat GPT writes requirement spec |
| **Branch** | Execution context | `feature/task-42-auth-service` |
| **Pull Request** | Proposed work | Cursor's implementation for review |
| **Commit** | Atomic change | `feat: implement JWT (fixes #42)` |
| **Labels** | Routing signals | `assigned/cursor`, `priority/high` |
| **Comments** | Decision history | Why certain choices were made |
| **Project Board** | State visualization | Backlog → Ready → In Progress → Review → Done |

---

## Task Lifecycle

```
┌─────────────────────────────────────────────────────────┐
│ 1. CHATGPT CREATES TASK SPEC (Issue)                    │
│    - Fills out task-spec.md template                    │
│    - Includes success criteria, context, metadata        │
│    - Labels: orchestration/task-spec, status/todo        │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 2. GITHUB ROUTES TASK                                   │
│    - GitHub Action triggers (task-router.yml)           │
│    - Creates branch: feature/task-{issue}-{slug}        │
│    - Labels issue: assigned/cursor, status/todo         │
│    - Comments: "Ready for Cursor to implement"          │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 3. CURSOR IMPLEMENTS TASK                               │
│    - Reads issue #XXX for requirements                  │
│    - Checks out branch: feature/task-XXX-*              │
│    - Implements code changes                            │
│    - Commits with message: "feat: X (fixes #XXX)"       │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 4. CURSOR OPENS PULL REQUEST                            │
│    - Uses PR template                                   │
│    - Links to original issue: "Closes #XXX"             │
│    - Lists what changed, how to test                    │
│    - Checks: criteria met, tests pass, docs updated     │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 5. GITHUB VALIDATES PR                                  │
│    - GitHub Action: cursor-validation.yml               │
│    - Checks PR format, linked issue, test pass          │
│    - Posts ✅ or ❌ validation result                   │
│    - Labels issue: status/review                        │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 6. BASE44 DASHBOARD SHOWS                               │
│    - Task moved to "In Review"                          │
│    - Team sees PR waiting for approval                  │
│    - Can see validation status                          │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 7. HUMAN REVIEWS & APPROVES                             │
│    - Reviews code in GitHub                             │
│    - Approves PR                                        │
│    - Merges to main                                     │
└────────────────────┬────────────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────────────┐
│ 8. GITHUB FINALIZES                                     │
│    - GitHub Action: integration-sync.yml                │
│    - Updates original issue: label status/done          │
│    - Notifies Base44 webhook (if configured)            │
│    - Deletes feature branch (preserves in commit)       │
│    - Archives in history                                │
└─────────────────────────────────────────────────────────┘
```

---

## Repository Structure

```
orchestrator-repo/
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   └── task-spec.md              # ChatGPT creates these
│   ├── PULL_REQUEST_TEMPLATE.md      # Standard Cursor output format
│   ├── workflows/
│   │   ├── task-router.yml           # Routes issues to Cursor
│   │   ├── cursor-validation.yml     # Validates PR completeness
│   │   └── integration-sync.yml      # Syncs with Base44/external
│   └── task-router-config.yml        # Label taxonomy
│
├── docs/
│   ├── ORCHESTRATION.md              # This file
│   ├── WORKFLOW.md                   # Day-to-day operations
│   └── API-CONTRACTS.md              # System interfaces
│
├── src/                              # Actual deliverable code
│   ├── orchestrator/
│   └── ...
│
├── tasks/                            # Optional: Task specs as files
│   └── 2026-q1/
│
├── testing/                          # Validation scripts
│   └── task-validation/
│
└── README.md                         # Getting started
```

---

## Label Strategy

### Status Labels (Choose One Per Issue)
- `status/todo` — Task ready, waiting for implementation
- `status/in-progress` — Cursor actively working
- `status/review` — PR open, awaiting review
- `status/blocked` — Waiting on external input
- `status/done` — Merged and complete

### Routing Labels (Indicate Who Should Act)
- `assigned/cursor` — Cursor's action required
- `assigned/openai` — OpenAI should refine spec
- `assigned/human` — Manual intervention needed
- `assigned/base44` — Base44 workflow needed

### Metadata Labels (Optional But Useful)
- **Priority**: `priority/critical`, `priority/high`, `priority/medium`, `priority/low`
- **Effort**: `effort/xs`, `effort/small`, `effort/medium`, `effort/large`
- **Type**: `type/feature`, `type/bugfix`, `type/refactor`, `type/test`
- **Component**: `component/core`, `component/integrations`, `component/api-layer`

---

## GitHub Actions Workflows

### 1. task-router.yml (Automatic)
Triggers when new task spec issue created

**Actions:**
- Creates feature branch: `feature/task-{issue}-{slug}`
- Labels issue: `assigned/cursor`, `status/todo`
- Comments: "Ready for implementation"

### 2. cursor-validation.yml (Automatic)
Triggers when PR opened/updated

**Validates:**
- PR uses template (has all required sections)
- PR links to issue (`Closes #XXX`)
- Tests pass (if repo has tests)
- No merge conflicts

**Posts:** ✅ or ❌ result

### 3. integration-sync.yml (Automatic)
Triggers when PR merged to main

**Actions:**
- Updates original issue: adds `status/done` label
- Notifies Base44 webhook (if configured)
- Deletes feature branch
- Creates comment in original issue

---

## Integration Points

### ChatGPT/OpenAI Integration
**ChatGPT writes:** Issues using task-spec.md template
- Title format: `[TASK] Brief description`
- Required fields: Executive Summary, Success Criteria, Context
- Metadata: Component, Effort, Type, Priority
- Result: Issue created → GitHub Action routes to Cursor

### Cursor Integration
**Cursor reads:** Issues and branches
**Cursor writes:** Commits and PRs

```bash
# Cursor workflow
1. Read issue #42 (task spec)
2. git checkout feature/task-42-auth-service
3. Implement code changes
4. git commit -m "feat: implement JWT (fixes #42)"
5. Open PR with linked issue
6. Watch for validation results
```

**Cursor does NOT:**
- Write directly to main/develop
- Merge PRs (waits for human approval)
- Delete branches (GitHub Actions handles cleanup)
- Create deployment triggers (manual gate)

### Base44 Integration
**Base44 reads:** Issues, PRs, labels
**Base44 writes:** Only via GitHub API if webhook enabled

```json
// Base44 receives webhook on task completion:
{
  "event": "task.completed",
  "issue": 42,
  "pr": 156,
  "merged_at": "2026-03-19T15:30:00Z"
}
```

**Base44 does NOT:**
- Directly commit code
- Approve/merge PRs
- Delete or rename branches
- Override GitHub decisions

---

## MVP Phase (Weeks 1-3)

### Week 1: Foundation
- [x] Repository created
- [x] Issue template: task-spec.md
- [x] PR template configured
- [x] Labels defined and colored
- [x] Branch protection rules set
- [x] Documented in README

### Week 2: Manual Loop
- [x] ChatGPT manually creates issues
- [x] Cursor manually creates branches and PRs
- [x] Humans manually approve PRs on GitHub
- [x] All via GitHub web UI or CLI
- ⏳ No GitHub Actions yet
- ⏳ No Base44 integration yet
- ⏳ No MCP yet

### Week 3: First Automation
- [x] GitHub Action: Auto-create branch on issue (task-router.yml)
- [x] GitHub Action: Validate PR format (cursor-validation.yml)
- [x] GitHub Action: Update issue on PR merge (integration-sync.yml)
- [x] Project board auto-moves columns
- ⏳ Cursor still opens PRs manually
- ⏳ Base44 reads only (dashboard)
- ⏳ Webhooks optional

---

## Phase 2+: Advanced Features

- Cursor auto-creates PRs via API
- Auto-merge for trivial changes
- MCP servers for richer context
- ChatGPT refines unclear specs automatically
- Dependency graph and blocking detection
- Performance metrics and velocity tracking
- Incident response automation

---

## Non-Conflict Rules (System Boundaries)

1. **Cursor owns code branches**
   - Only Cursor pushes to `feature/task-*`
   - Base44 never directly commits
   - Base44 can comment/approve only

2. **Base44 owns UI view**
   - Base44 reads via API
   - Base44 renders dashboard
   - GitHub remains source of truth

3. **GitHub is commit-of-record**
   - Merge to main = task complete (immutable)
   - Full history preserved
   - Base44 syncs state FROM GitHub, not vice versa

4. **Decision authority**
   - ChatGPT/OpenAI → Task spec authority
   - Cursor → Code implementation authority
   - Base44 → Presentation authority
   - GitHub → State/history authority
   - Humans → Tie-breaker authority

---

## FAQ

**Q: What if Cursor finds a spec is unclear?**  
A: Cursor comments on the issue with questions. OpenAI responds in comments. Issue stays open until clarified.

**Q: What if a PR has conflicts?**  
A: Validation workflow fails. Cursor is notified to rebase. PR remains open until resolved.

**Q: What if a human wants to override a decision?**  
A: Override via comment + label change. Full decision history is preserved with timestamps.

**Q: Can multiple systems work on the same branch?**  
A: No. One-issue-one-branch rule. Create separate issues for parallel work.

**Q: Is MCP required?**  
A: No. MVP uses only GitHub API + webhooks. MCP adds context richness later.

---

Last Updated: 2026-03-19
