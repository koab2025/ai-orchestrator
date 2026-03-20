# AI Orchestrator

A GitHub-based orchestration system coordinating OpenAI, Cursor, Base44, and humans in a contract-first development workflow.

## 🎯 Core Concept

**GitHub is the single source of truth.**

- **OpenAI/ChatGPT** writes task specifications as GitHub issues
- **Cursor** reads specs and implements code changes via git branches and PRs
- **Base44** monitors progress via GitHub API and webhooks
- **Humans** review and approve merges
- **GitHub** maintains complete auditability and history

## 🚀 Quick Start

### 1. Create a Task (ChatGPT/OpenAI)

Go to Issues → New Issue → Select "Task Spec (ChatGPT)" template

Required fields:
- Title: `[TASK] Brief description`
- Executive Summary
- Success Criteria (checkboxes)
- Context for Cursor
- Metadata (Component, Effort, Type, Priority)

**Result:** GitHub automatically creates a `feature/task-{id}-{slug}` branch and labels the issue `assigned/cursor`.

### 2. Implement a Task (Cursor)

```bash
git checkout feature/task-42-auth-service
# ... make changes ...
git commit -m "feat: implement JWT auth (fixes #42)"
git push
```

Open PR using GitHub's "Compare & pull request" button.

**Result:** GitHub Actions validate your PR against success criteria.

### 3. Review & Merge (Human)

Review the PR in GitHub. If approved, merge.

**Result:** Issue automatically marked `status/done`, branch deleted, Base44 notified.

## 📋 Key Files

- [`.github/ISSUE_TEMPLATE/task-spec.md`](.github/ISSUE_TEMPLATE/task-spec.md) — Issue template for task specs
- [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md) — PR template for implementations
- [`.github/workflows/task-router.yml`](.github/workflows/task-router.yml) — Auto-creates branch on new task
- [`.github/workflows/cursor-validation.yml`](.github/workflows/cursor-validation.yml) — Validates PR completeness
- [`.github/workflows/integration-sync.yml`](.github/workflows/integration-sync.yml) — Syncs with Base44
- [`docs/ORCHESTRATION.md`](docs/ORCHESTRATION.md) — Architecture details
- [`docs/WORKFLOW.md`](docs/WORKFLOW.md) — Day-to-day operations guide

## 🏗️ Architecture

```
OpenAI                 Cursor                 Base44
  |                      |                      |
  |-- Create Issue ----> GitHub <---- Read API |
  |                      |                      |
  |                   Create Branch             |
  |                   Commit Code               |
  |                   Open PR                   |
  |                      |                      |
  |                   GitHub Action             |
  |                   Validate PR               |
  |                      |                      |
  |<--- Review via PR --- | ---- Webhook ------->|
  |                      |                      |
  |                   Merge to main             |
  |                      |                      |
  |           Update Issue & Notify             |
  |                      |                      |
  |----- History Preserved ---- Full Audit Log |
```

## 📊 Status Lifecycle

```
todo (OpenAI creates)
  ↓
in-progress (Cursor works)
  ↓
review (Cursor opens PR)
  ↓
(Human reviews)
  ↓
done (Human merges)
```

**Labels drive the workflow:** Each status is a GitHub label, automatically updated by Actions or manually by team.

## 🎯 MVP Phase (Weeks 1-3)

### Week 1: Foundation
- [x] Repository created
- [x] Templates in place
- [x] Labels defined
- [x] Branch protection on main

### Week 2: Manual Operations
- [x] OpenAI manually creates issues
- [x] Cursor manually creates branches and PRs
- [x] Humans manually merge
- [x] All via GitHub UI or CLI

### Week 3: Automation
- [x] GitHub Actions auto-create branches
- [x] GitHub Actions auto-validate PRs
- [x] GitHub Actions auto-update on merge
- [x] Project board auto-moves

## 🚀 Phase 2+: Advanced Features

- Cursor API integration (auto-creates PRs)
- MCP servers (richer context)
- Auto-merge for trivial changes
- ChatGPT auto-refines specs
- Dependency graphs
- Velocity metrics
- Incident automation

## 🤝 Integration Points

### ChatGPT/OpenAI
- **Reads:** Nothing (creates issues manually)
- **Writes:** Issues as task specs

### Cursor
- **Reads:** Issues (task specs), branches
- **Writes:** Commits, PRs
- **Does NOT:** Merge, delete branches, approve PRs

### Base44
- **Reads:** Issues, PRs, labels via API
- **Writes:** Optional webhook to receive task completion
- **Does NOT:** Commit code, merge, override GitHub

### Humans
- **Reads:** PRs, issues in GitHub
- **Writes:** PR approvals, merges
- **Decision gate:** Final approval before production

## 📚 Documentation

- [`docs/ORCHESTRATION.md`](docs/ORCHESTRATION.md) — Full architecture & design
- [`docs/WORKFLOW.md`](docs/WORKFLOW.md) — Day-to-day operations
- [GitHub Issues](../../issues) — The actual task backlog

## 🔒 Non-Conflict Rules

1. **One issue = one branch** → prevents parallel conflicts
2. **Cursor owns code** → only Cursor can commit to feature branches
3. **Humans own merge gate** → approval required before production
4. **Base44 reads-only** → integrates but doesn't override GitHub
5. **GitHub is source of truth** → all decisions recorded as commits

## 🛠️ Setting Up Locally

```bash
git clone <repo>
cd orchestrator
git checkout main

# View issues
# github.com/you/repo/issues

# Create a task (if you're OpenAI/ChatGPT)
# Go to Issues → New Issue → Task Spec template

# Start implementing (if you're Cursor)
git checkout feature/task-42-auth-service
# ... make changes ...
git commit -m "feat: implement X (fixes #42)"
git push
```

## 📞 Support

- **Issue unclear?** Comment on the issue, tag `@openai`
- **PR validation failing?** Check GitHub Action comments, fix and push again
- **PR stuck in review?** Comment to ask for feedback
- **Something broken?** Create an issue with `type/bugfix`

## 📝 License

See LICENSE file (or add your own).

---

**Last Updated:** 2026-03-19  
**Status:** MVP Phase — Ready for human-in-the-loop operations
