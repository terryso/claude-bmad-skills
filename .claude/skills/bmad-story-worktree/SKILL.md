---
description: Complete BMAD user story delivery in isolated worktree, merge only after tests pass
argument-hint: <story-number> e.g., 1.1 or 2.3 (optional, auto-selects smallest backlog story if omitted)
---

# BMAD Story Deliver (Worktree Edition)

Complete the full delivery pipeline for user story `{ARGUMENT}` in an isolated git worktree, merge to main branch only after all tests pass.

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

## Difference from bmad-story-deliver

| Feature | bmad-story-deliver | bmad-story-worktree |
|---------|-------------------|---------------------|
| Working method | Develop on current branch | Develop in isolated worktree |
| Code isolation | None | Complete isolation |
| Merge condition | None enforced | Tests pass + fixes complete |
| Safety level | Medium | High |

## Execution Strategy

Use **Task tool** to execute each phase sequentially with visible progress:

1. Launch independent `general-purpose` agent for each phase
2. Agent returns structured results upon completion
3. Main session outputs progress bars

---

## Execution Flow

### Step 1/8: Create Worktree

Launch Task agent to create isolated worktree using `git worktree`:

```
Task(
  subagent_type: general-purpose,
  description: "Create worktree {ARGUMENT}",
  prompt: "Create isolated worktree for story {ARGUMENT} using git worktree. Steps:

1. Get current project name, path and branch:
   ORIGINAL_REPO_PATH=$(pwd)
   PROJECT_NAME=$(basename $(pwd))
   CURRENT_BRANCH=$(git branch --show-current)

2. Create new branch name: feature/story-{ARGUMENT}

3. Create worktree directory (in sibling directory):
   WORKTREE_PATH=\"../${PROJECT_NAME}-story-{ARGUMENT}\"

4. Execute git worktree command:
   git worktree add -b feature/story-{ARGUMENT} \"$WORKTREE_PATH\" $CURRENT_BRANCH

   If branch already exists, use:
   git worktree add \"$WORKTREE_PATH\" feature/story-{ARGUMENT}

5. Verify worktree created:
   git worktree list

6. Return: worktree absolute path, branch name, base branch, original repo path"
)
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [1/8] Create Worktree
   🌿 Branch: feature/story-{ARGUMENT}
   📁 Path: ../{project-name}-story-{ARGUMENT}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 2/8: Create User Story

Launch Task agent to execute creation (in worktree):

```
Task(
  subagent_type: general-purpose,
  description: "Create user story {ARGUMENT}",
  prompt: "Execute /bmad-bmm-create-story {ARGUMENT} in worktree to create or update user story. Return: 1) Story ID 2) Title 3) List of created/updated files"
)
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [2/8] Create User Story
   📝 Story: {ARGUMENT}
   📄 Files: [file list]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 3/8: Development

Launch Task agent to execute development (in worktree):

```
Task(
  subagent_type: general-purpose,
  description: "Develop user story {ARGUMENT}",
  prompt: "Execute /bmad-bmm-dev-story {ARGUMENT} in worktree to implement user story code. Return: 1) List of modified files 2) Summary of changes 3) Any issues to note"
)
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [3/8] Development
   📁 Modified files: [count]
   🔧 Changes: [summary]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 4/8: QA Automated Testing

Launch Task agent to execute testing (in worktree):

```
Task(
  subagent_type: general-purpose,
  description: "QA test user story {ARGUMENT}",
  prompt: "Execute /bmad-bmm-qa-automate {ARGUMENT} in worktree to generate and run automated tests. Return: 1) Test file list 2) Test pass rate 3) Failed cases (if any)"
)
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [4/8] QA Automated Testing
   🧪 Tests: [passed/total]
   ⚡ Status: [All passed / Has failures]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 5/8: Code Review

Launch Task agent to execute review (in worktree):

```
Task(
  subagent_type: general-purpose,
  description: "Review user story {ARGUMENT}",
  prompt: "Execute /bmad-bmm-code-review {ARGUMENT} in worktree to review code changes. Return: 1) Review conclusion (pass/needs-fix) 2) Issues found (categorized by severity: HIGH/MEDIUM/LOW) 3) Improvement suggestions"
)
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [5/8] Code Review
   👀 Conclusion: [pass/needs-fix]
   🔴 HIGH: [count]
   🟡 MEDIUM: [count]
   🔵 LOW: [count]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 6/8: Auto-fix Issues

**Only executed if HIGH or MEDIUM issues are found during code review.**

Launch Task agent to auto-fix issues (in worktree):

```
Task(
  subagent_type: general-purpose,
  description: "Fix review issues {ARGUMENT}",
  prompt: "Based on code review results, auto-fix all HIGH and MEDIUM severity issues in worktree. Fix principles:
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
✅ [6/8] Auto-fix
   🔧 Fixed: [HIGH + MEDIUM count] issues
   📁 Modified files: [count]
   🧪 Tests: [pass/fail]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**If no HIGH/MEDIUM issues, skip this step:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏭️ [6/8] Auto-fix
   ✨ No HIGH/MEDIUM issues, skipping fix step
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 7/8: Merge or Preserve

**Critical decision step: Check merge conditions**

Merge conditions:
1. ✅ All QA tests pass
2. ✅ No HIGH/MEDIUM issues (or all fixed)
3. ✅ Tests still pass after fixes

**If all conditions are met, execute merge:**

```
Task(
  subagent_type: general-purpose,
  description: "Merge story branch {ARGUMENT}",
  prompt: "Execute merge and cleanup using git worktree:

1. Commit all changes in worktree:
   cd {WORKTREE_PATH}
   git add .
   git commit -m \"feat: complete story {ARGUMENT}\"

2. Switch back to main repo:
   cd {ORIGINAL_REPO_PATH}

3. Merge feature branch:
   git merge feature/story-{ARGUMENT} --no-edit

4. Remove worktree:
   git worktree remove {WORKTREE_PATH}

5. Optionally delete feature branch (if not needed):
   git branch -d feature/story-{ARGUMENT}

6. Verify cleanup:
   git worktree list

7. Return: merge status, commit hash, worktree cleanup status"
)
```

**Progress output:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ [7/8] Merge Branch
   🔀 Merge: feature/story-{ARGUMENT} → [current branch]
   🗑️ Cleanup: worktree removed
   📝 Commit: [hash]
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**If merge conditions not met, preserve worktree for manual handling:**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️ [7/8] Merge Branch
   ❌ Merge conditions not met, preserving worktree
   📁 Path: {WORKTREE_PATH}
   🔧 Manual handling needed:
   - [List unmet conditions]

   After handling, manually execute:
   cd {WORKTREE_PATH}
   git add . && git commit -m "fix: resolve issues"
   cd {ORIGINAL_REPO_PATH}
   git merge feature/story-{ARGUMENT}
   git worktree remove {WORKTREE_PATH}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Step 8/8: Update Status to Done

**After successful merge, must update both files.**

#### 8.1 Update sprint-status.yaml

Use Edit tool to update status file:

```
File path: _bmad-output/implementation-artifacts/sprint-status.yaml

Change story status from ready-for-dev or in-progress to done:
  {epic-num}-{story-num}-{story-name}: done
```

#### 8.2 Update Story Design Document

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
✅ [8/8] Update Status
   📝 sprint-status.yaml: {ARGUMENT} → done
   📄 Story doc: Status: done, Tasks: ✅
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Final Delivery Report

**Successful merge:**
```
╔════════════════════════════════════════════════════════╗
║           🎉 BMAD Story Delivery Complete!             ║
╠════════════════════════════════════════════════════════╣
║  Story: {ARGUMENT}                                     ║
║                                                        ║
║  ✅ Step 1 - Create Worktree                           ║
║  ✅ Step 2 - Create User Story                         ║
║  ✅ Step 3 - Development                               ║
║  ✅ Step 4 - QA Automated Testing                      ║
║  ✅ Step 5 - Code Review                               ║
║  ✅ Step 6 - Auto-fix Issues                           ║
║  ✅ Step 7 - Merge Branch                              ║
║  ✅ Step 8 - Update Status to Done                     ║
║                                                        ║
║  📊 Status: done                                       ║
╚════════════════════════════════════════════════════════╝
```

**Manual intervention required:**
```
╔════════════════════════════════════════════════════════╗
║        ⚠️ BMAD Story Delivery - Manual Intervention    ║
╠════════════════════════════════════════════════════════╣
║  Story: {ARGUMENT}                                     ║
║                                                        ║
║  ✅ Step 1 - Create Worktree                           ║
║  ✅ Step 2 - Create User Story                         ║
║  ⚠️ Step 3 - Development [has issues]                  ║
║  ...                                                    ║
║  ❌ Step 7 - Merge Branch [conditions not met]         ║
║                                                        ║
║  📊 Status: Awaiting manual handling                   ║
║  📁 Worktree: {WORKTREE_PATH}                          ║
╚════════════════════════════════════════════════════════╝
```

---

## Error Handling

If any step fails:
1. Stop executing subsequent steps
2. Preserve worktree (don't delete)
3. Output error information and current state
4. Prompt user how to manually fix
5. Provide recovery commands

**Recovery command examples:**
```bash
# List all worktrees
git worktree list

# Continue working in worktree
cd {WORKTREE_PATH}

# Commit after manual completion
git add . && git commit -m "fix: manual fix"

# Switch back to main repo and merge
cd {ORIGINAL_REPO_PATH}
git merge feature/story-{ARGUMENT}

# Cleanup worktree
git worktree remove {WORKTREE_PATH}

# Optional: delete feature branch
git branch -d feature/story-{ARGUMENT}
```
