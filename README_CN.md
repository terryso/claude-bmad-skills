# Claude BMAD Skills

BMAD (Breakthrough Method of Agile AI Driven Development) 方法论的 Claude Code Skills 集合。

[English](README.md)

## 安装

将所需的 skill 复制到你的 Claude Code skills 目录：

```bash
# 复制单个 skill
cp -r .claude/skills/bmad-story-deliver ~/.claude/skills/

# 或复制所有 skills
cp -r .claude/skills/* ~/.claude/skills/
```

## 可用 Skills

### 技能总览

| 技能 | 执行模式 | 管道 | 隔离 |
|------|---------|------|------|
| bmad-story-pipeline | 子代理 | 可配置 | 无 |
| bmad-story-pipeline-worktree | 子代理 | 可配置 | Worktree |
| bmad-story-deliver | 子代理 | 固定 6 步 | 无 |
| bmad-story-worktree | 子代理 | 固定 8 步 | Worktree |
| bmad-story-team-deliver | 代理团队 | 固定 5 步 | 上下文 |
| bmad-epic-pipeline-worktree | 批量 | 可配置 | Worktree |
| bmad-epic-worktree | 批量 | 固定 | Worktree |

> 💡 **推荐**：使用 `bmad-story-pipeline` 或 `bmad-story-pipeline-worktree`，支持可配置工作流。

---

### bmad-story-pipeline

使用子代理运行可配置的 BMAD 管道交付用户故事。

```bash
/bmad-story-pipeline 1-1
# 或不传参数，自动选择
/bmad-story-pipeline
```

**特性：**
- 通过 `references/workflow-steps.md` 配置工作流
- 每个步骤在独立子代理中运行（全新上下文）
- 可自定义管道步骤

**默认管道：**
1. 创建用户故事
2. 生成 ATDD 测试
3. 开发实现
4. 代码审查
5. 追踪测试覆盖

---

### bmad-story-pipeline-worktree

在独立 worktree 中运行可配置的 BMAD 管道，测试通过后才合并。

```bash
/bmad-story-pipeline-worktree 1-1
# 或不传参数，自动选择
/bmad-story-pipeline-worktree
```

**特性：**
- 包含 `bmad-story-pipeline` 的所有特性
- 额外的 worktree 隔离
- 条件合并（测试通过 + 无 HIGH/MEDIUM 问题）

**与 bmad-story-pipeline 的区别：**
| 特性 | pipeline | pipeline-worktree |
|------|----------|-------------------|
| 工作方式 | 当前分支 | 独立 worktree |
| 代码隔离 | 无 | 完全隔离 |
| 合并条件 | 无强制要求 | 测试通过才合并 |
| 安全性 | 中 | 高 |

---

### bmad-story-deliver

完成 BMAD 用户故事的完整交付流程（创建 → 开发 → QA → 审查 → 自动修复 → 更新状态）。

```bash
/bmad-story-deliver 1.1
# 或不传参数，自动选择编号最小的 backlog 故事
/bmad-story-deliver
```

**流程步骤：**
1. 创建用户故事
2. 开发实现
3. QA 自动化测试
4. 代码审查
5. 自动修复 HIGH/MEDIUM 问题
6. 更新状态为 Done

---

### bmad-story-worktree

在独立 git worktree 中完成用户故事交付，所有测试通过后才合并到主分支。

```bash
/bmad-story-worktree 1.1
# 或不传参数，自动选择编号最小的 backlog 故事
/bmad-story-worktree
```

**流程步骤：**
1. 创建 Worktree（隔离开发环境）
2. 创建用户故事
3. 开发实现
4. QA 自动化测试
5. 代码审查
6. 自动修复 HIGH/MEDIUM 问题
7. 合并分支（仅当测试通过 + 无遗留问题）
8. 更新状态为 Done

**与 deliver 版的区别：**
| 特性 | deliver | worktree |
|------|---------|----------|
| 代码隔离 | 无 | 完全隔离 |
| 合并条件 | 无强制要求 | 测试通过才合并 |
| 安全性 | 中 | 高 |

---

### bmad-story-team-deliver

使用代理团队运行 BMAD 管道，每个步骤使用独立上下文。

```bash
/bmad-story-team-deliver 1-1
# 或不传参数，自动选择
/bmad-story-team-deliver
```

**特性：**
- 每个管道步骤在专属队友中运行
- 每步全新上下文（无上下文污染）
- 复杂工作流的团队协作

---

### bmad-epic-pipeline-worktree

使用可配置管道在独立 worktree 中交付整个 Epic。

```bash
/bmad-epic-pipeline-worktree 3
# 或不传参数，自动选择
/bmad-epic-pipeline-worktree
```

**特性：**
- 与 `bmad-epic-worktree` 相同，但使用 `bmad-story-pipeline-worktree`
- 通过 workflow-steps.md 配置管道
- 每个故事独立 worktree 隔离

---

### bmad-epic-worktree

交付整个 Epic，逐个完成其中所有未完成的用户故事。

```bash
/bmad-epic-worktree 3
# 或不传参数，自动选择编号最小且有未完成故事的 Epic
/bmad-epic-worktree
```

**执行逻辑：**
1. 收集 Epic 下所有未完成的故事
2. 按 Story 编号升序排序
3. 逐个调用 `/bmad-story-worktree` 交付
4. 前一个故事完成才开始下一个
5. 任一失败则停止，保留状态

**适用场景：**
- 批量交付整个 Epic
- 自动化多故事顺序开发
- 确保每个故事独立测试通过

---

## 目录结构

```
claude-bmad-skills/
├── README.md
├── README_CN.md
├── LICENSE
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
│       ├── bmad-epic-worktree/
│       │   └── SKILL.md
│       └── bmad-epic-pipeline-worktree/
│           └── SKILL.md
```

## 如何选择合适的 Skill

**单个故事交付：**
- 🌟 **推荐**：`bmad-story-pipeline` 或 `bmad-story-pipeline-worktree`（可配置工作流）
- 需要 worktree 隔离？ → `bmad-story-pipeline-worktree`
- 简单快速？ → `bmad-story-deliver` 或 `bmad-story-worktree`

**整个 Epic 交付：**
- 🌟 **推荐**：`bmad-epic-pipeline-worktree`（可配置工作流）
- 固定管道？ → `bmad-epic-worktree`

## 贡献

欢迎提交 Issue 和 Pull Request 添加新的 BMAD skills。

## License

MIT
