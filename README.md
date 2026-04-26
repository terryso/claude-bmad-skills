# Claude BMAD Skills

Claude Code Skills collection for BMAD (Breakthrough Method of Agile AI Driven Development) methodology.

[中文文档](README_CN.md)

## Installation

If you want a Chinese place to search and install skills, check out Skills宝: https://skilery.com

### Option 1: Claude Code Plugin Marketplace (Recommended)

```bash
/plugin marketplace add terryso/claude-bmad-skills
/plugin install bmad-skills
```

### Option 2: npx skills

```bash
npx skills add terryso/claude-bmad-skills
```

### Option 3: Git Submodule

```bash
git submodule add https://github.com/terryso/claude-bmad-skills.git .agents/bmad-skills
```

### Option 4: Clone & Copy

```bash
git clone https://github.com/terryso/claude-bmad-skills.git
cp -r claude-bmad-skills/skills/* ~/.claude/skills/
```

## Available Skills

### Skill Overview

| Skill | Execution Mode | Pipeline | Isolation |
|-------|---------------|----------|-----------|
| bmad-story-pipeline | Subagent | Configurable | None |
| bmad-story-pipeline-worktree | Subagent | Configurable | Worktree |
| bmad-epic-pipeline-worktree | Batch | Configurable | Worktree |

> 💡 **Recommended**: Use `bmad-story-pipeline` or `bmad-story-pipeline-worktree` for configurable workflow.

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

### bmad-epic-pipeline-worktree

Deliver entire Epic using configurable pipeline in isolated worktrees.

```bash
/bmad-epic-pipeline-worktree 3
# Or omit argument to auto-select
/bmad-epic-pipeline-worktree
```

**Features:**
- Uses `bmad-story-pipeline-worktree` for each story
- Configurable pipeline via workflow-steps.md
- Worktree isolation per story

**Execution logic:**
1. Collect all incomplete stories under Epic
2. Sort by Story number ascending
3. Call `/bmad-story-pipeline-worktree` for each story sequentially
4. Next story starts only after previous completes
5. If any fails, stop and preserve state

**Use cases:**
- Batch deliver entire Epic
- Automate multi-story sequential development
- Ensure each story passes tests independently

---

## Directory Structure

```
claude-bmad-skills/
├── README.md
├── README_CN.md
├── LICENSE
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
└── skills/
    ├── bmad-story-pipeline/
    │   ├── SKILL.md
    │   └── references/workflow-steps.md
    ├── bmad-story-pipeline-worktree/
    │   ├── SKILL.md
    │   └── references/workflow-steps.md
    └── bmad-epic-pipeline-worktree/
        └── SKILL.md
```

## Choosing the Right Skill

**For single story:**
- 🌟 **Recommended**: `bmad-story-pipeline` or `bmad-story-pipeline-worktree` (configurable workflow)
- Need worktree isolation? → `bmad-story-pipeline-worktree`

**For entire Epic:**
- 🌟 **Recommended**: `bmad-epic-pipeline-worktree` (configurable workflow)

## Contributing

Issues and Pull Requests welcome to add new BMAD skills.

## License

MIT
