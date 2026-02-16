---
description: Complete BMAD user story delivery pipeline (Create → Develop → QA → Review → Auto-fix)
argument-hint: <story-number> e.g., 1.1 or 2.3 (optional, auto-selects smallest backlog story if omitted)
---

# BMAD Story Deliver

Complete the full delivery pipeline for user story `{ARGUMENT}`.

## Pre-step: Determine Story Number

**If no story number is provided (`{ARGUMENT}` is empty):**

1. Read `_bmad-output/implementation-artifacts/sprint-status.yaml`
2. Find all stories with status `backlog` (format: `X-Y-story-name`)
3. Sort by Epic number X ascending, then by Story number Y ascending
4. Select the story with the smallest number as `{ARGUMENT}`
5. Output:
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
📌 Auto-selected story: {ARGUMENT} (status: backlog)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Example conversion:**
- Key in status file: `3-4-prediction-result-storage` → Story number: `3.4`
- Key in status file: `4-2-thread-safe-state-management` → Story number: `4.2`

---

## Execution Strategy

Use **Task tool** to execute each phase sequentially with visible progress:

1. Launch independent `general-purpose` agent for each phase
2. Agent returns structured results upon completion
3. Main session outputs progress bars

---

## Execution Flow

### Step 1/6: Create User Story

Launch Task agent to execute creation:

```
Task(
  subagent_type: general-purpose,
  description: "Create user story {ARGUMENT}",
  prompt: "Execute /bmad-bmm-create-story {ARGUMENT} to create or update user story. Return: 1) Story ID 2) Title 3) List of created/updated files"
)
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [1/6] Create User Story
   📝 Story: {ARGUMENT}
   📄 Files: [file list]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 2/6: Development

Launch Task agent to execute development:

```
Task(
  subagent_type: general-purpose,
  description: "Develop user story {ARGUMENT}",
  prompt: "Execute /bmad-bmm-dev-story {ARGUMENT} to implement user story code. Return: 1) List of modified files 2) Summary of changes 3) Any issues to note"
)
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [2/6] Development
   📁 Modified files: [count]
   🔧 Changes: [summary]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 3/6: QA Automated Testing

Launch Task agent to execute testing:

```
Task(
  subagent_type: general-purpose,
  description: "QA test user story {ARGUMENT}",
  prompt: "Execute /bmad-bmm-qa-automate {ARGUMENT} to generate and run automated tests. Return: 1) Test file list 2) Test pass rate 3) Failed cases (if any)"
)
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [3/6] QA Automated Testing
   🧪 Tests: [passed/total]
   ⚡ Status: [All passed / Has failures]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 4/6: Code Review

Launch Task agent to execute review:

```
Task(
  subagent_type: general-purpose,
  description: "Review user story {ARGUMENT}",
  prompt: "Execute /bmad-bmm-code-review {ARGUMENT} to review code changes. Return: 1) Review conclusion (pass/needs-fix) 2) Issues found (categorized by severity: HIGH/MEDIUM/LOW) 3) Improvement suggestions"
)
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [4/6] Code Review
   👀 Conclusion: [pass/needs-fix]
   🔴 HIGH: [count]
   🟡 MEDIUM: [count]
   🔵 LOW: [count]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 5/6: Auto-fix Issues

**Only executed if HIGH or MEDIUM issues are found during code review.**

Launch Task agent to auto-fix issues:

```
Task(
  subagent_type: general-purpose,
  description: "Fix review issues {ARGUMENT}",
  prompt: "Based on code review results, auto-fix all HIGH and MEDIUM severity issues. Fix principles:
1. Fix HIGH level issues first
2. Then fix MEDIUM level issues
3. Document LOW level issues in code comments, no immediate fix needed
4. Re-run related tests after fix to ensure no regression
5. Return: list of fixed issues, modified files, test results"
)
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [5/6] Auto-fix
   🔧 Fixed: [HIGH + MEDIUM count] issues
   📁 Modified files: [count]
   🧪 Tests: [pass/fail]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**If no HIGH/MEDIUM issues, skip this step:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏭️ [5/6] Auto-fix
   ✨ No HIGH/MEDIUM issues, skipping fix step
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 6/6: Update Status to Done

**After successful delivery, must update both files.**

#### 6.1 Update sprint-status.yaml

Use Edit tool to update status file:

```
File path: _bmad-output/implementation-artifacts/sprint-status.yaml

Change story status from ready-for-dev or in-progress to done:
  {epic-num}-{story-num}-{story-name}: done
```

#### 6.2 Update Story Design Document

Use Edit tool to update story document status:

```
File path: _bmad-output/implementation-artifacts/{epic-num}-{story-num}-{story-name}.md

Update two places:

1. Top status line:
   Change "Status: ready-for-dev" or "Status: in-progress" to "Status: done"

2. Tasks checklist (if any):
   Change all "- [ ]" to "- [x]" to mark tasks complete
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [6/6] Update Status
   📝 sprint-status.yaml: {ARGUMENT} → done
   📄 Story doc: Status: done, Tasks: ✅
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Final Delivery Report

On completion:

```
╔════════════════════════════════════════════════════════╗
║           🎉 BMAD Story Delivery Complete!             ║
╠════════════════════════════════════════════════════════╣
║  Story: {ARGUMENT}                                     ║
║                                                        ║
║  ✅ Step 1 - Create User Story                         ║
║  ✅ Step 2 - Development                               ║
║  ✅ Step 3 - QA Automated Testing                      ║
║  ✅ Step 4 - Code Review                               ║
║  ✅ Step 5 - Auto-fix Issues                           ║
║  ✅ Step 6 - Update Status to Done                     ║
║                                                        ║
║  📊 Status: done                                       ║
╚════════════════════════════════════════════════════════╝
```

---

## Error Handling

If any step fails:
1. Stop executing subsequent steps
2. Output error information
3. Prompt user how to fix
