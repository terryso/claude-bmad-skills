---
description: 在独立 worktree 中完成 BMAD 用户故事交付，测试通过后才合并
argument-hint: <故事编号> 例如: 1.1 或 2.3
---

# BMAD 故事交付 (Worktree 版)

在独立 git worktree 中完成用户故事 `{ARGUMENT}` 的完整交付流程，所有测试通过后才合并到主分支。

## 与 bmad-story-deliver 的区别

| 特性 | bmad-story-deliver | bmad-story-worktree |
|------|-------------------|---------------------|
| 工作方式 | 直接在当前分支开发 | 独立 worktree 隔离开发 |
| 代码隔离 | 无 | 完全隔离 |
| 合并条件 | 无强制要求 | 测试通过 + 修复完成 |
| 安全性 | 中 | 高 |

## 执行策略

使用 **Task 工具** 顺序执行每个阶段，确保进度可见：

1. 每个阶段启动独立的 `general-purpose` agent
2. Agent 完成后返回结构化结果
3. 主会话输出进度条

---

## 执行流程

### Step 1/7: 创建 Worktree

启动 Task agent 创建独立工作树：

```
Task(
  subagent_type: general-purpose,
  description: "创建 worktree {ARGUMENT}",
  prompt: "为故事 {ARGUMENT} 创建独立的 git worktree。步骤：
1. 从当前分支创建新分支 feature/story-{ARGUMENT}（如果已存在则使用现有分支）
2. 创建 worktree 目录 ../{项目名}-story-{ARGUMENT}
3. 切换到新 worktree
4. 返回：worktree 路径、分支名、创建状态"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [1/7] 创建 Worktree
   🌿 分支: feature/story-{ARGUMENT}
   📁 路径: ../{项目名}-story-{ARGUMENT}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 2/7: 创建用户故事

启动 Task agent 执行创建任务（在 worktree 中）：

```
Task(
  subagent_type: general-purpose,
  description: "创建用户故事 {ARGUMENT}",
  prompt: "在 worktree 中执行 /bmad-bmm-create-story {ARGUMENT}，创建或更新用户故事。完成后返回：1) 故事ID 2) 标题 3) 创建/更新的文件列表"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [2/7] 创建用户故事
   📝 故事: {ARGUMENT}
   📄 文件: [文件列表]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 3/7: 开发实现

启动 Task agent 执行开发（在 worktree 中）：

```
Task(
  subagent_type: general-purpose,
  description: "开发用户故事 {ARGUMENT}",
  prompt: "在 worktree 中执行 /bmad-bmm-dev-story {ARGUMENT}，实现用户故事的功能代码。完成后返回：1) 修改的文件列表 2) 主要改动摘要 3) 是否有需要关注的问题"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [3/7] 开发实现
   📁 修改文件: [数量] 个
   🔧 改动: [摘要]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 4/7: QA 自动化测试

启动 Task agent 执行测试（在 worktree 中）：

```
Task(
  subagent_type: general-purpose,
  description: "QA测试用户故事 {ARGUMENT}",
  prompt: "在 worktree 中执行 /bmad-bmm-qa-automate {ARGUMENT}，生成并运行自动化测试。完成后返回：1) 测试文件列表 2) 测试通过率 3) 失败用例（如有）"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [4/7] QA 自动化测试
   🧪 测试: [通过数/总数]
   ⚡ 状态: [全部通过 / 有失败]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 5/7: 代码审查

启动 Task agent 执行审查（在 worktree 中）：

```
Task(
  subagent_type: general-purpose,
  description: "审查用户故事 {ARGUMENT}",
  prompt: "在 worktree 中执行 /bmad-bmm-code-review {ARGUMENT}，审查本故事的代码变更。完成后返回：1) 审查结论（通过/需修改）2) 发现的问题（按严重程度分类：HIGH/MEDIUM/LOW）3) 改进建议"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [5/7] 代码审查
   👀 结论: [通过/需修改]
   🔴 HIGH: [数量] 个
   🟡 MEDIUM: [数量] 个
   🔵 LOW: [数量] 个
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 6/7: 自动修复问题

**仅在代码审查发现 HIGH 或 MEDIUM 问题时执行此步骤。**

启动 Task agent 自动修复问题（在 worktree 中）：

```
Task(
  subagent_type: general-purpose,
  description: "修复审查问题 {ARGUMENT}",
  prompt: "根据上一步代码审查的结果，在 worktree 中自动修复所有 HIGH 和 MEDIUM 严重程度的问题。修复原则：
1. 优先修复 HIGH 级别问题
2. 然后修复 MEDIUM 级别问题
3. LOW 级别问题在代码注释中记录，不需要立即修复
4. 修复后重新运行相关测试确保不引入回归
5. 完成后返回：已修复的问题列表、修改的文件、测试结果"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [6/7] 自动修复
   🔧 已修复: [HIGH数 + MEDIUM数] 个问题
   📁 修改文件: [数量] 个
   🧪 测试: [通过/失败]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**如果代码审查无 HIGH/MEDIUM 问题，跳过此步骤并输出：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏭️ [6/7] 自动修复
   ✨ 无 HIGH/MEDIUM 问题，跳过修复步骤
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 7/7: 合并或保留

**关键决策步骤：检查是否满足合并条件**

合并条件：
1. ✅ QA 测试全部通过
2. ✅ 无 HIGH/MEDIUM 级别问题（或已修复）
3. ✅ 修复后测试仍通过

**如果满足所有条件，执行合并：**

```
Task(
  subagent_type: general-purpose,
  description: "合并故事分支 {ARGUMENT}",
  prompt: "执行合并操作：
1. 在 worktree 中提交所有更改
2. 切换回主仓库
3. 合并 feature/story-{ARGUMENT} 分支到当前分支
4. 删除 worktree
5. 可选：删除 feature 分支
6. 返回：合并状态、提交哈希"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [7/7] 合并分支
   🔀 合并: feature/story-{ARGUMENT} → [当前分支]
   🗑️ 清理: worktree 已删除
   📝 提交: [哈希]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**如果不满足合并条件，保留 worktree 等待人工处理：**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━���━━━━━━━━━
⚠️ [7/7] 合并分支
   ❌ 不满足合并条件，保留 worktree
   📁 路径: ../{项目名}-story-{ARGUMENT}
   🔧 需要人工处理:
   - [列出未满足的条件]

   处理完成后手动执行:
   git merge feature/story-{ARGUMENT}
   git worktree remove ../{项目名}-story-{ARGUMENT}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 最终交付报告

全部完成后输出：

**成功合并：**
```
╔════════════════════════════════════════════════════════╗
║           🎉 BMAD 故事交付完成！                        ║
╠════════════════════════════════════════════════════════╣
║  故事编号: {ARGUMENT}                                   ║
║                                                        ║
║  ✅ Step 1 - 创建 Worktree                              ║
║  ✅ Step 2 - 创建用户故事                               ║
║  ✅ Step 3 - 开发实现                                   ║
║  ✅ Step 4 - QA 自动化测试                              ║
║  ✅ Step 5 - 代码审查                                   ║
║  ✅ Step 6 - 自动修复问题                               ║
║  ✅ Step 7 - 合并分支                                   ║
║                                                        ║
║  📊 状态: 交付成功 ✨                                    ║
╚════════════════════════════════════════════════════════╝
```

**需要人工介入：**
```
╔════════════════════════════════════════════════════════╗
║        ⚠️ BMAD 故事交付 - 需要人工介入                  ║
╠════════════════════════════════════════════════════════╣
║  故事编号: {ARGUMENT}                                   ║
║                                                        ║
║  ✅ Step 1 - 创建 Worktree                              ║
║  ✅ Step 2 - 创建用户故事                               ║
║  ⚠️ Step 3 - 开发实现 [有问题]                          ║
║  ...                                                    ║
║  ❌ Step 7 - 合并分支 [未满足条件]                       ║
║                                                        ║
║  📊 状态: 等待人工处理                                   ║
║  📁 Worktree: ../{项目名}-story-{ARGUMENT}              ║
╚════════════════════════════════════════════════════════╝
```

---

## 错误处理

如果任何步骤失败：
1. 停止执行后续步骤
2. 保留 worktree 不删除
3. 输出错误信息和当前状态
4. 提示用户如何手动修复
5. 提供恢复命令

**恢复命令示例：**
```bash
# 继续在 worktree 中工作
cd ../{项目名}-story-{ARGUMENT}

# 手动完成后合并
git add . && git commit -m "fix: 手动修复问题"
cd {原项目目录}
git merge feature/story-{ARGUMENT}

# 清理 worktree
git worktree remove ../{项目名}-story-{ARGUMENT}
```
