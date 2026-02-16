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

完成 BMAD 用户故事的完整交付流程（创建 → 开发 → QA → 审查 → 自动修复）。

```bash
/bmad-story-deliver 1.1
```

**流程步骤：**
1. 创建用户故事
2. 开发实现
3. QA 自动化测试
4. 代码审查
5. 自动修复问题（HIGH/MEDIUM 级别）

## 目录结构

```
claude-bmad-skills/
├── README.md
├── .claude/
│   └── skills/
│       └── bmad-story-deliver/
│           └── SKILL.md
└── LICENSE
```

## 贡献

欢迎提交 Issue 和 Pull Request 添加新的 BMAD skills。

## License

MIT
