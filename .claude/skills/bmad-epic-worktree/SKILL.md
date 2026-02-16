---
description: 交付整个 Epic，逐个完成其中所有未完成的用户故事
argument-hint: <Epic编号> 例如: 1 或 2（可选，不传则自动选择编号最小且有未完成故事的 Epic）
---

# BMAD Epic 交付 (Worktree 版)

交付整个 Epic `{ARGUMENT}` 中所有未完成的用户故事，每个故事使用独立 worktree 开发，测试通过后才合并。

## 前置步骤：确定 Epic 编号

**如果调用时未传入 Epic 编号 `{ARGUMENT}` 为空：**

1. 读取 `_bmad-output/implementation-artifacts/sprint-status.yaml`
2. 查找所有状态不为 `done` 的故事（格式：`X-Y-story-name`）
3. 提取这些故事的 Epic 编号 X
4. 选择最小的 Epic 编号作为 `{ARGUMENT}`
5. 输出提示信息：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📌 自动选择 Epic: {ARGUMENT} (有未完成的故事)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**示例：**
- 状态文件中 `3-2-llm-prompt-template: backlog`, `3-3-xxx: in-progress`, `4-1-xxx: done`
- 未完成的 Epic 有: 3
- 自动选择 Epic: 3

---

## 执行策略

1. 读取 Epic 下所有未完成的故事
2. 按 Story 编号升序排序
3. **逐个**执行 `/bmad-story-worktree` 交付每个故事
4. 前一个故事完成后才开始下一个
5. 任一故事失败则停止，保留当前状态

---

## 执行流程

### Step 1: 收集 Epic 故事列表

读取 sprint-status.yaml，收集指定 Epic 下所有未完成的故事：

```
Task(
  subagent_type: general-purpose,
  description: "收集 Epic {ARGUMENT} 故事列表",
  prompt: "读取 _bmad-output/implementation-artifacts/sprint-status.yaml，收集 Epic {ARGUMENT} 的所有故事：

1. 筛选 key 格式为 '{ARGUMENT}-Y-story-name' 的条目
2. 只保留状态不为 'done' 的故事
3. 按 Story 编号 Y 升序排序
4. 返回故事列表，格式：
   - 故事编号: '1.1', '1.2', ...
   - 故事名称
   - 当前状态

如果没有未完成的故事，返回空列表并提示 'Epic {ARGUMENT} 所有故事已完成'"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📋 [1/?] 收集故事列表
   📁 Epic: {ARGUMENT}
   📝 未完成故事: [数量] 个
   🔢 顺序: {story-1}, {story-2}, ...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 2~N: 逐个交付故事

**对每个未完成的故事，按顺序执行：**

```
对于故事 {STORY_NUM} (第 i 个，共 N 个):

执行 /bmad-story-worktree {STORY_NUM}

等待完成后继续下一个故事
```

**每个故事交付进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔄 故事 [{i}/{N}]: {STORY_NUM}
   📝 名称: {story-name}
   ⏳ 执行中...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[... 执行 bmad-story-worktree 的 8 个步骤 ...]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ 故事 [{i}/{N}]: {STORY_NUM} 完成
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**如果某个故事失败：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ 故事 [{i}/{N}]: {STORY_NUM} 失败
   ⚠️ 停止后续故事交付
   📁 请手动处理后再继续
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 最终交付报告

**全部完成：**
```
╔════════════════════════════════════════════════════════╗
║           🎉 BMAD Epic 交付完成！                       ║
╠════════════════════════════════════════════════════════╣
║  Epic 编号: {ARGUMENT}                                  ║
║                                                        ║
║  ✅ 故事 1: {story-1} - done                            ║
║  ✅ 故事 2: {story-2} - done                            ║
║  ...                                                    ║
║  ✅ 故事 N: {story-N} - done                            ║
║                                                        ║
║  📊 总计: {N}/{N} 故事完成                              ║
║  🎯 Epic 状态: done                                     ║
╚════════════════════════════════════════════════════════╝
```

**部分完成（有失败）：**
```
╔════════════════════════════════════════════════════════╗
║        ⚠️ BMAD Epic 交付 - 部分完成                     ║
╠════════════════════════════════════════════════════════╣
║  Epic 编号: {ARGUMENT}                                  ║
║                                                        ║
║  ✅ 故事 1: {story-1} - done                            ║
║  ✅ 故事 2: {story-2} - done                            ║
║  ❌ 故事 3: {story-3} - 失败                            ║
║  ⏸️ 故事 4: {story-4} - 未开始                          ║
║  ...                                                    ║
║                                                        ║
║  📊 进度: {completed}/{total} 故事完成                  ║
║  📁 失败故事: {failed-story}                            ║
║  💡 处理失败故事后重新运行继续交付                       ║
╚════════════════════════════════════════════════════════╝
```

---

## 错误处理

如果任一故事交付失败：
1. 停止后续故事交付
2. 保留失败故事的 worktree（如有）
3. 输出失败信息和已完成的进度
4. 提示用户手动处理

**恢复继续交付：**
```bash
# 1. 手动修复失败的故事
cd {WORKTREE_PATH}
# 修复问题...
git add . && git commit -m "fix: 修复问题"
cd {ORIGINAL_REPO_PATH}
git merge feature/story-{STORY_NUM}
git worktree remove {WORKTREE_PATH}

# 2. 重新运行 epic 交付，会自动跳过已完成的故事
/bmad-epic-worktree {ARGUMENT}
```

---

## 与 bmad-story-worktree 的关系

- **bmad-epic-worktree** 是 **bmad-story-worktree** 的批量版本
- 内部循环调用 `/bmad-story-worktree {story-num}`
- 每个故事独立 worktree，独立测试，独立合并
- 保证顺序执行，前一个完成才开始下一个
