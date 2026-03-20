# Tasks (Optional)

This directory optionally stores task specifications as structured files rather than only in GitHub Issues.

## When to Use This

- For archival (backup) of task specs
- For off-GitHub searches
- For version control of task definitions
- For complex/multi-step tasks with long context

## When NOT to Use This

- For MVP phase (use GitHub Issues only)
- If it creates duplication
- If it's just extra work without benefit

## Format

```
tasks/
├── 2026-q1/
│   ├── 001-feature-auth.md
│   ├── 002-bugfix-timeout.md
│   └── 003-refactor-api.md
└── 2026-q2/
    └── ...
```

## File Format

Each task file should reference its GitHub issue:

```markdown
# Feature: User Authentication Service

**Issue:** #42  
**Status:** Ready for Cursor  
**Component:** API Layer  
**Effort:** Medium  

## Summary
Implement JWT-based authentication service...

## Success Criteria
- [ ] Criterion 1
- [ ] Criterion 2

(Rest of spec mirrors GitHub issue)
```

## Synchronization

- **Source of truth:** GitHub Issues
- **Backup/Archive:** This directory
- Keep these in sync manually or via GitHub Actions (Phase 2+)

## Recommendations

For MVP: Skip this directory. Use GitHub Issues only.
When adding: Keep it minimal. GitHub is still source of truth.
