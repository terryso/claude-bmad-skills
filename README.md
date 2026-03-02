# Claude BMAD Skills

Claude Code Skills collection for BMAD (Breakthrough Method of Agile AI Driven Development) methodology.

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

### Skill Overview

| Skill | Execution Mode | Pipeline | Isolation |
|-------|---------------|----------|-----------|
| bmad-story-deliver | Direct | Fixed 6-step | None |
| bmad-story-worktree | Direct | Fixed 8-step | Worktree |
| bmad-story-pipeline | Subagent | Configurable | None |
| bmad-story-pipeline-worktree | Subagent | Configurable | Worktree |
| bmad-story-team-deliver | Agent Team | Fixed 5-step | Context |
| bmad-story-team-worktree | Agent Team | Fixed 5-step | Worktree |
| bmad-epic-worktree | Batch | Fixed | Worktree |
| bmad-epic-pipeline-worktree | Batch | Configurable | Worktree |

---

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

### bmad-story-pipeline

Run configurable BMAD pipeline for story delivery using subagent.

```bash
/bmad-story-pipeline 1-1
# Or omit argument to auto-select
/bmad-story-pipeline
```

**Features:**
- Configurable workflow via `references/workflow-steps.md`
- Each step runs in isolated subagent (fresh context)
- Customizable pipeline steps

**Default pipeline:**
1. Create user story
2. Generate ATDD tests
3. Development
4. Code review
5. Trace test coverage

---

### bmad-story-pipeline-worktree

Run configurable BMAD pipeline in isolated worktree, merge only after tests pass.

```bash
/bmad-story-pipeline-worktree 1-1
# Or omit argument to auto-select
/bmad-story-pipeline-worktree
```

**Features:**
- All features of `bmad-story-pipeline`
- Plus worktree isolation
- Conditional merge (tests pass + no HIGH/MEDIUM issues)

**Difference from bmad-story-pipeline:**
| Feature | pipeline | pipeline-worktree |
|---------|----------|-------------------|
| Working method | Current branch | Isolated worktree |
| Code isolation | None | Complete isolation |
| Merge condition | None enforced | Tests pass + fixes complete |
| Safety level | Medium | High |

---

### bmad-story-team-deliver

Run BMAD pipeline using agent teams with isolated context per step.

```bash
/bmad-story-team-deliver 1-1
# Or omit argument to auto-select
/bmad-story-team-deliver
```

**Features:**
- Each pipeline step runs in dedicated teammate
- Fresh context per step (no context pollution)
- Team coordination for complex workflows

---

### bmad-story-team-worktree

Complete BMAD user story delivery using agent teams in isolated worktree, merge only after tests pass.

```bash
/bmad-story-team-worktree 1-1
# Or omit argument to auto-select
/bmad-story-team-worktree
```

**Features:**
- All features of `bmad-story-team-deliver`
- Plus worktree isolation
- Conditional merge

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

---

### bmad-epic-pipeline-worktree

Deliver entire Epic using configurable pipeline in isolated worktrees.

```bash
/bmad-epic-pipeline-worktree 3
# Or omit argument to auto-select
/bmad-epic-pipeline-worktree
```

**Features:**
- Same as `bmad-epic-worktree` but uses `bmad-story-pipeline-worktree`
- Configurable pipeline via workflow-steps.md
- Worktree isolation per story

---

## Directory Structure

```
claude-bmad-skills/
├── README.md
├── README_CN.md
├── .claude/
│   └── skills/
│       ├── bmad-story-deliver/
│       │   └── SKILL.md
│       ├── bmad-story-worktree/
│       │   └── SKILL.md
│       ├── bmad-story-pipeline/
│       │   ├── SKILL.md
│       │   └── references/workflow-steps.md
│       ├── bmad-story-pipeline-worktree/
│       │   ├── SKILL.md
│       │   └── references/workflow-steps.md
│       ├── bmad-story-team-deliver/
│       │   └── SKILL.md
│       ├── bmad-story-team-worktree/
│       │   └── SKILL.md
│       ├── bmad-epic-worktree/
│       │   └── SKILL.md
│       └── bmad-epic-pipeline-worktree/
│           └── SKILL.md
└── LICENSE
```

## Choosing the Right Skill

**For single story:**
- Need isolation? → `bmad-story-worktree` or `bmad-story-pipeline-worktree`
- Want configurable pipeline? → `bmad-story-pipeline` or `bmad-story-pipeline-worktree`
- Want agent teams? → `bmad-story-team-deliver` or `bmad-story-team-worktree`
- Simple & fast? → `bmad-story-deliver`

**For entire Epic:**
- Fixed pipeline? → `bmad-epic-worktree`
- Configurable pipeline? → `bmad-epic-pipeline-worktree`

## Contributing

Issues and Pull Requests welcome to add new BMAD skills.

## License

MIT
