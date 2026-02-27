---
description: Run BMAD pipeline using agent teams with isolated context per step
argument-hint: <story-number> e.g., 1-1 or 2-3 (optional, auto-selects smallest backlog story if omitted)
---

# BMAD Story Delivery (Agent Team Edition)

Run the complete BMAD pipeline for story `{ARGUMENT}` using agent teams.

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
- Key in status file: `3-4-prediction-result-storage` → Story number: `3-4`
- Key in status file: `4-2-thread-safe-state-management` → Story number: `4-2`

Set `STORY_ID = {ARGUMENT}` for all subsequent steps.

---

## Execution Instructions

Create an agent team to run the BMAD pipeline for story `${STORY_ID}`.
Each step MUST run in its own teammate (fresh context) and execute
strictly in sequence. Do not start the next step until the previous
one completes successfully. If any step fails, stop the entire pipeline
immediately and report which step failed and why.

The pipeline steps with strict dependencies:

Step 1: /bmad-bmm-create-story ${STORY_ID} yolo
  → Creates the story definition. No dependencies.

Step 2: /bmad-tea-testarch-atdd ${STORY_ID} yolo
  → Generates ATDD test architecture. Depends on Step 1.

Step 3: /bmad-bmm-dev-story ${STORY_ID} yolo
  → Develops the story implementation. Depends on Step 2.

Step 4: /bmad-bmm-code-review ${STORY_ID} yolo
  → Runs code review on the implementation. Depends on Step 3.

Step 5: /bmad-tea-testarch-trace ${STORY_ID} yolo
  → Traces test architecture coverage. Depends on Step 4.

Rules:
- Spawn ONE teammate per step, only after the previous step completes.
- Each teammate runs its slash command with "yolo" mode (auto-approve).
- Do NOT run any steps in parallel.
- Act as coordinator only (delegate mode). Do not execute steps yourself.
- After each step completes, confirm success before spawning the next teammate.
- On failure: report the failed step, the error, and stop. Do not continue.
- After all 5 steps complete, provide a summary of the full pipeline run.
