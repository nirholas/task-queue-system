# Prompt 01: Sample Task

## Context
You are setting up this repository template. Create a basic validation file to prove the task-runner flow works end-to-end.

## Rules
- Work ONLY in the current repository
- Stay on branch `main`
- ALWAYS use `isBackground: true` for terminal commands and ALWAYS kill terminals after getting output
- Commit and push as `nirholas`
- No mocks, no stubs, no placeholder code

## File Ownership
- **Creates**: `example.txt`

## Task
1. Create `example.txt` with exact content:
   - `Task queue system is working.`

## Verification
```bash
cat example.txt
# Task queue system is working.
```

## Success Criteria
- `example.txt` exists
- Content exactly matches the required text
