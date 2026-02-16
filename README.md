# Claude BMAD Skills

BMAD (Behavior-Model-Artifact-Document) 方法论的 Claude Code Skills 集合。

## 安装

将所需的 skill 复制到你的 Claude Code skills 目录：

```bash
# 复制单个 skill
cp -r .claude/skills/bmad-story-deliver ~/.claude/skills/

# 或复制所有 skills
cp -r .claude/skills/* ~/.claude/skills/
```

## 可用 Skills

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
5. 自动修复问题（HIGH/MEDIUM 级别）
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
6. 自动修复问题（HIGH/MEDIUM 级别）
7. 合并分支（仅当测试通过 + 无遗留问题）
8. 更新状态为 Done

**与 deliver 版的区别：**
| 特性 | deliver | worktree |
|------|---------|----------|
| 代码隔离 | 无 | 完全隔离 |
| 合并条件 | 无 | 测试通过才合并 |
| 安全性 | 中 | 高 |

## 目录结构

```
claude-bmad-skills/
├── README.md
├── .claude/
│   └── skills/
│       ├── bmad-story-deliver/
│       │   └── SKILL.md
│       └── bmad-story-worktree/
│           └── SKILL.md
└── LICENSE
```

## 贡献

欢迎提交 Issue 和 Pull Request 添加新的 BMAD skills。

## License

MIT
