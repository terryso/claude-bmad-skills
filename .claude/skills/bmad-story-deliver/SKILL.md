---
description: 完成 BMAD 用户故事的完整交付流程（创建 → 开发 → QA → 审查 → 自动修复）
argument-hint: <故事编号> 例如: 1.1 或 2.3（可选，不传则自动选择编号最小的 backlog 故事）
---

# BMAD 故事交付 (Story Deliver)

完成用户故事 `{ARGUMENT}` 的完整交付流程。

## 前置步骤：确定故事编号

**如果调用时未传入故事编号 `{ARGUMENT}` 为空：**

1. 读取 `_bmad-output/implementation-artifacts/sprint-status.yaml`
2. 查找所有状态为 `backlog` 的故事（格式：`X-Y-story-name`）
3. 按 Epic 编号 X 升序，再按 Story 编号 Y 升序排序
4. 选择编号最小的故事作为 `{ARGUMENT}`
5. 输出提示信息：
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📌 自动选择故事: {ARGUMENT} (状态: backlog)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**示例转换：**
- 状态文件中的 key: `3-4-prediction-result-storage` → 故事编号: `3.4`
- 状态文件中的 key: `4-2-thread-safe-state-management` → 故事编号: `4.2`

---

## 执行策略

使用 **Task 工具** 顺序执行每个阶段，确保进度可见：

1. 每个阶段启动独立的 `general-purpose` agent
2. Agent 完成后返回结构化结果
3. 主会话输出进度条

---

## 执行流程

### Step 1/5: 创建用户故事

启动 Task agent 执行创建任务：

```
Task(
  subagent_type: general-purpose,
  description: "创建用户故事 {ARGUMENT}",
  prompt: "执行 /bmad-bmm-create-story {ARGUMENT}，创建或更新用户故事。完成后返回：1) 故事ID 2) 标题 3) 创建/更新的文件列表"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [1/5] 创建用户故事
   📝 故事: {ARGUMENT}
   📄 文件: [文件列表]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 2/5: 开发实现

启动 Task agent 执行开发：

```
Task(
  subagent_type: general-purpose,
  description: "开发用户故事 {ARGUMENT}",
  prompt: "执行 /bmad-bmm-dev-story {ARGUMENT}，实现用户故事的功能代码。完成后返回：1) 修改的文件列表 2) 主要改动摘要 3) 是否有需要关注的问题"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [2/5] 开发实现
   📁 修改文件: [数量] 个
   🔧 改动: [摘要]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 3/5: QA 自动化测试

启动 Task agent 执行测试：

```
Task(
  subagent_type: general-purpose,
  description: "QA测试用户故事 {ARGUMENT}",
  prompt: "执行 /bmad-bmm-qa-automate {ARGUMENT}，生成并运行自动化测试。完成后返回：1) 测试文件列表 2) 测试通过率 3) 失败用例（如有）"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [3/5] QA 自动化测试
   🧪 测试: [通过数/总数]
   ⚡ 状态: [全部通过 / 有失败]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 4/5: 代码审查

启动 Task agent 执行审查：

```
Task(
  subagent_type: general-purpose,
  description: "审查用户故事 {ARGUMENT}",
  prompt: "执行 /bmad-bmm-code-review {ARGUMENT}，审查本故事的代码变更。完成后返回：1) 审查结论（通过/需修改）2) 发现的问题（按严重程度分类：HIGH/MEDIUM/LOW）3) 改进建议"
)
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [4/5] 代码审查
   👀 结论: [通过/需修改]
   🔴 HIGH: [数量] 个
   🟡 MEDIUM: [数量] 个
   🔵 LOW: [数量] 个
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 5/6: 自动修复问题

**仅在代码审查发现 HIGH 或 MEDIUM 问题时执行此步骤。**

启动 Task agent 自动修复问题：

```
Task(
  subagent_type: general-purpose,
  description: "修复审查问题 {ARGUMENT}",
  prompt: "根据上一步代码审查的结果，自动修复所有 HIGH 和 MEDIUM 严重程度的问题。修复原则：
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
✅ [5/6] 自动修复
   🔧 已修复: [HIGH数 + MEDIUM数] 个问题
   📁 修改文件: [数量] 个
   🧪 测试: [通过/失败]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**如果代码审查无 HIGH/MEDIUM 问题，跳过此步骤并输出：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏭️ [5/6] 自动修复
   ✨ 无 HIGH/MEDIUM 问题，跳过修复步骤
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 6/6: 更新状态为 Done

**交付成功后，必须同时更新两个文件的状态。**

#### 6.1 更新 sprint-status.yaml

使用 Edit 工具更新状态文件：

```
文件路径: _bmad-output/implementation-artifacts/sprint-status.yaml

将故事状态从 ready-for-dev 或 in-progress 改为 done：
  {epic-num}-{story-num}-{story-name}: done
```

#### 6.2 更新故事详细设计文档

使用 Edit 工具更新故事文档状态：

```
文件路径: _bmad-output/implementation-artifacts/{epic-num}-{story-num}-{story-name}.md

需要更新两处：

1. 顶部状态行：
   将 "Status: ready-for-dev" 或 "Status: in-progress" 改为 "Status: done"

2. Tasks 子任务清单（如果有）：
   将所有 "- [ ]" 改为 "- [x]" 表示任务完成
```

**输出进度：**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [6/6] 更新状态
   📝 sprint-status.yaml: {ARGUMENT} → done
   📄 故事文档: Status: done, Tasks: ✅
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 最终交付报告

全部完成后输出：

```
╔════════════════════════════════════════════════════════╗
║           🎉 BMAD 故事交付完成！                        ║
╠════════════════════════════════════════════════════════╣
║  故事编号: {ARGUMENT}                                   ║
║                                                        ║
║  ✅ Step 1 - 创建用户故事                               ║
║  ✅ Step 2 - 开发实现                                   ║
║  ✅ Step 3 - QA 自动化测试                              ║
║  ✅ Step 4 - 代码审查                                   ║
║  ✅ Step 5 - 自动修复问题                               ║
║  ✅ Step 6 - 更新状态为 Done                            ║
║                                                        ║
║  📊 状态: done                                          ║
╚════════════════════════════════════════════════════════╝
```

---

## 错误处理

如果任何步骤失败：
1. 停止执行后续步骤
2. 输出错误信息
3. 提示用户如何修复
