# Claude BMAD Skills

Claude Code Skills collection for BMAD (Behavior-Model-Artifact-Document) methodology.

[中文文档](README_CN.md)

## Installation

Copy the desired skills to your Claude Code skills directory:

```bash
# Copy a single skill
cp -r .claude/skills/bmad-story-deliver ~/.claude/skills/

# Or copy all skills
cp -r .claude/skills/* ~/.claude/skills/
```

## Available Skills

### bmad-story-deliver

Complete BMAD user story delivery pipeline (Create → Develop → QA → Review → Auto-fix → Update status).

```bash
/bmad-story-deliver 1.1
# Or omit argument to auto-select smallest backlog story
/bmad-story-deliver
```

**Pipeline steps:**
1. Create user story
2. Development
3. QA automated testing
4. Code review
5. Auto-fix HIGH/MEDIUM issues
6. Update status to Done

---

### bmad-story-worktree

Complete user story delivery in isolated git worktree, merge only after all tests pass.

```bash
/bmad-story-worktree 1.1
# Or omit argument to auto-select smallest backlog story
/bmad-story-worktree
```

**Pipeline steps:**
1. Create Worktree (isolated dev environment)
2. Create user story
3. Development
4. QA automated testing
5. Code review
6. Auto-fix HIGH/MEDIUM issues
7. Merge branch (only when tests pass + no outstanding issues)
8. Update status to Done

**Difference from deliver version:**
| Feature | deliver | worktree |
|---------|---------|----------|
| Code isolation | None | Complete isolation |
| Merge condition | None enforced | Tests pass + fixes complete |
| Safety level | Medium | High |

---

### bmad-epic-worktree

Deliver entire Epic by completing all incomplete user stories sequentially.

```bash
/bmad-epic-worktree 3
# Or omit argument to auto-select smallest epic with incomplete stories
/bmad-epic-worktree
```

**Execution logic:**
1. Collect all incomplete stories under Epic
2. Sort by Story number ascending
3. Call `/bmad-story-worktree` for each story sequentially
4. Next story starts only after previous completes
5. If any fails, stop and preserve state

**Use cases:**
- Batch deliver entire Epic
- Automate multi-story sequential development
- Ensure each story passes tests independently

## Directory Structure

```
claude-bmad-skills/
├── README.md
├── .claude/
│   └── skills/
│       ├── bmad-story-deliver/
│       │   └── SKILL.md
│       ├── bmad-story-worktree/
│       │   └── SKILL.md
│       └── bmad-epic-worktree/
│           └── SKILL.md
└── LICENSE
```

## Contributing

Issues and Pull Requests welcome to add new BMAD skills.

## License

MIT
