# BMAD Story Pipeline Workflow Steps

## Pipeline Configuration

Execute the following steps in order using Task tool.
Each step runs with "yolo" mode for auto-approval.

### Step 1: Create User Story
- Command: `/bmad-bmm-create-story {STORY_ID} yolo`
- Description: Creates story file with context from planning docs
- Return: Story ID, Title, Created files

### Step 2: Generate ATDD Tests
- Command: `/bmad-tea-testarch-atdd {STORY_ID} yolo`
- Description: Generate failing acceptance tests (TDD red phase)
- Return: ATDD checklist and test files

### Step 3: Development
- Command: `/bmad-bmm-dev-story {STORY_ID} yolo`
- Description: Implement story to pass tests (TDD green phase)
- Return: Modified files, Changes summary

### Step 4: Code Review
- Command: `/bmad-bmm-code-review {STORY_ID} yolo`
- Description: Adversarial code review for issues
- Return: Conclusion (pass/needs-fix), Issues by severity

### Step 5: Trace Test Coverage
- Command: `/bmad-tea-testarch-trace {STORY_ID} yolo`
- Description: Generate traceability matrix and quality gate decision
- Return: Coverage percentage, Gate decision

## Post-Pipeline

After all steps complete:
1. Update sprint-status.yaml: story status → done
2. Update story document: Status: done, Tasks: ✅

## Customization

To modify the pipeline:
- Add/remove steps in this file
- Change step commands
- Reorder steps (update step numbers)
