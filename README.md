# Task Queue System — Portable Installation Guide

> **Copy this system to any Git repo to enable numbered task dispatch across 50+ parallel agent chats.**

---

## What This System Does

Enables you to:
1. Break large work into individual task `.md` files
2. Number them in a manifest
3. Open 50 VS Code Copilot chats
4. Type `@task-runner 1` through `@task-runner 50`
5. Each agent reads its task file and executes it completely

**Zero manual copy-paste. Zero coordination. Fully parallelized execution.**

---

## Installation (5 Minutes)

### 1. Copy Core Files

From the `swarms` repo (or wherever you have this system):

```bash
# In your target repo
mkdir -p .github/agents

# Copy the 4 core files
cp /path/to/swarms/.github/agents/task-runner.agent.md .github/agents/
cp /path/to/swarms/.github/agents/prompt-writer.agent.md .github/agents/
cp /path/to/swarms/PROMPT-SPEC.md .
cp /path/to/swarms/TASK-QUEUE-SYSTEM.md .  # Optional: documentation
```

### 2. Create Your Prompts Directory

```bash
mkdir -p prompts
```

Create task files in `prompts/<category>/NN-<slug>.md` following the format in `PROMPT-SPEC.md`.

**Example structure**:
```
prompts/
  frontend/
    01-dashboard-layout.md
    02-responsive-design.md
    03-dark-mode.md
  backend/
    01-api-routes.md
    02-database-schema.md
    03-websocket-server.md
  tests/
    01-unit-tests.md
    02-integration-tests.md
```

### 3. Generate TASK-MANIFEST.md

Run this command in your repo root:

```bash
echo '# Task Manifest

> Open a chat, type `@task-runner 42` (any number below). Agent reads the file and executes it.

| # | Category | File |
|---|----------|------|' > TASK-MANIFEST.md

find prompts/ -name '*.md' ! -name 'README.md' ! -path '*/archived/*' 2>/dev/null | \
  sort | \
  awk 'BEGIN{n=1} {
    split($0, a, "/");
    cat=a[2];
    print "|", n++, "|", cat, "|", $0, "|"
  }' >> TASK-MANIFEST.md
```

This scans your `prompts/` directory and creates a numbered index.

### 4. Verify

```bash
# Should show 2 agent files
ls -la .github/agents/

# Should show all 4 core files
ls -la PROMPT-SPEC.md TASK-MANIFEST.md TASK-QUEUE-SYSTEM.md

# Should show your prompts
find prompts/ -name '*.md' | head -10
```

### 5. Commit

```bash
git add .github/ prompts/ PROMPT-SPEC.md TASK-MANIFEST.md TASK-QUEUE-SYSTEM.md
git commit -m "feat: add task queue system for parallel agent dispatch"
git push
```

---

## Usage in New Repo

### Execute Tasks

Open VS Code Copilot Chat in the new repo:

```
@task-runner 1
@task-runner 2
...
```

The agent reads `TASK-MANIFEST.md`, finds the file path, reads the prompt, executes it.

### Create New Tasks

```
@prompt-writer Build a complete authentication system with OAuth, JWT, and MFA
```

The agent creates individual `.md` files, appends to `TASK-MANIFEST.md`, commits.

---

## Customizing for Your Repo

### Update task-runner.agent.md

If your repo uses different commands/tools, edit `.github/agents/task-runner.agent.md`:

```markdown
### Workflow
...
7. Run `npm run typecheck` after TypeScript changes.  # <-- customize this
8. Run `npm test` to verify.                          # <-- and this
...
```

### Update PROMPT-SPEC.md

If your repo has different conventions (e.g., Python instead of TypeScript), update `PROMPT-SPEC.md`:

```markdown
## Required Sections
...
## Verification
Exact commands to verify the task is complete:
```bash
poetry run pytest              # <-- customize for your tech stack
poetry run mypy .
poetry run black --check .
```
...
```

### Add Repo-Specific Rules

In `PROMPT-SPEC.md`, add a "Repo Conventions" section:

```markdown
## Repo Conventions

- Use `pnpm` instead of `npm`
- Format with `biome` instead of `prettier`
- All API calls use `ky` instead of `fetch`
- Tests live in `__tests__/` not `tests/`
```

The `@prompt-writer` agent reads this file before creating new prompts, so it will follow your conventions.

---

## Example: Porting to `demo` Repo

```bash
cd /workspaces/demo

# Copy core files
mkdir -p .github/agents
cp /workspaces/swarms/.github/agents/*.md .github/agents/
cp /workspaces/swarms/PROMPT-SPEC.md .
cp /workspaces/swarms/TASK-QUEUE-SYSTEM.md .

# You might already have prompts in demo - generate manifest
echo '# Task Manifest

> Open a chat, type `@task-runner 42` (any number below). Agent reads the file and executes it.

| # | Category | File |
|---|----------|------|' > TASK-MANIFEST.md

find docs/tasks missions .agents/tasks -name '*.md' ! -name 'README.md' 2>/dev/null | \
  sort | \
  awk 'BEGIN{n=1} {
    split($0, a, "/");
    cat=a[2];
    print "|", n++, "|", cat, "|", $0, "|"
  }' >> TASK-MANIFEST.md

# Commit
git add .github/ PROMPT-SPEC.md TASK-MANIFEST.md TASK-QUEUE-SYSTEM.md
git commit -m "feat: add task queue system"
git push

# Usage
# @task-runner 1
```

---

## Regenerating the Manifest

If you add/remove prompt files manually, regenerate `TASK-MANIFEST.md`:

```bash
echo '# Task Manifest

> Open a chat, type `@task-runner 42` (any number below). Agent reads the file and executes it.

| # | Category | File |
|---|----------|------|' > TASK-MANIFEST.md

# Adjust this glob to match YOUR prompt directories
find prompts/ docs/tasks missions -name '*.md' ! -name 'README.md' ! -path '*/archived/*' 2>/dev/null | \
  sort | \
  awk 'BEGIN{n=1} {
    split($0, a, "/");
    cat=a[2];
    print "|", n++, "|", cat, "|", $0, "|"
  }' >> TASK-MANIFEST.md

git add TASK-MANIFEST.md
git commit -m "chore: regenerate task manifest"
git push
```

---

## Standalone Repo (Optional)

To distribute this as a portable template repo:

```bash
mkdir task-queue-system
cd task-queue-system

# Initialize
git init
git branch -M main

# Copy all files
cp -r /workspaces/swarms/.github .
cp /workspaces/swarms/PROMPT-SPEC.md .
cp /workspaces/swarms/TASK-QUEUE-SYSTEM.md .
mv TASK-QUEUE-SYSTEM.md README.md  # Make it the main docs

# Create a sample prompts dir
mkdir -p prompts/example
cat > prompts/example/01-sample-task.md << 'EOF'
# Prompt 01: Sample Task

## Context
This is a sample task to demonstrate the format.

## Rules
- Work ONLY in the current directory
- Stay on branch main
- ALWAYS use isBackground: true for terminals, always kill_terminal after
- Commit as nirholas

## File Ownership
- **Creates**: `example.txt`

## Task
1. Create `example.txt` with content "Hello from the task queue system!"

## Verification
```bash
cat example.txt
# Should output: Hello from the task queue system!
```

## Success Criteria
- File `example.txt` exists
- Contains the expected text
EOF

# Generate manifest
echo '# Task Manifest

| # | Category | File |
|---|----------|------|
| 1 | example | prompts/example/01-sample-task.md |' > TASK-MANIFEST.md

# Commit
git add .
git commit -m "feat: initial task queue system template"

# Push to GitHub
gh repo create nirholas/task-queue-system --public --source=. --remote=origin
git push -u origin main
```

Then anyone can:

```bash
git clone https://github.com/nirholas/task-queue-system
cd task-queue-system
# Test: @task-runner 1
# Then replace prompts/example/ with their own tasks
```

---

## Key Files Reference

| File | Role | Required? |
|------|------|-----------|
| `.github/agents/task-runner.agent.md` | Executes tasks by number | ✅ Yes |
| `.github/agents/prompt-writer.agent.md` | Creates new tasks | ✅ Yes |
| `PROMPT-SPEC.md` | Format spec for prompts | ✅ Yes |
| `TASK-MANIFEST.md` | Numbered index | ✅ Yes |
| `TASK-QUEUE-SYSTEM.md` | Documentation | ⚪ Optional |
| `prompts/` | Your task files | ✅ Yes (or any dir) |

---

## Troubleshooting

### Agent doesn't recognize @task-runner

Reload VS Code: `Cmd+Shift+P` → "Reload Window"

VS Code Copilot scans `.github/agents/*.agent.md` on startup.

### Task manifest lookup fails

Verify the file path in `TASK-MANIFEST.md` matches the actual file:

```bash
# Extract task 42's file path
awk -F'|' '/^\| 42 /{gsub(/^ +| +$/,"",$4); print $4}' TASK-MANIFEST.md

# Check if it exists
ls -la "$(awk -F'|' '/^\| 42 /{gsub(/^ +| +$/,"",$4); print $4}' TASK-MANIFEST.md)"
```

### Agent creates prompts with wrong format

`@prompt-writer` reads `PROMPT-SPEC.md` before writing. If the spec is wrong, update it and re-run.

---

## Support & Contributing

This system was built in the `nirholas/swarms` repo. File issues or contribute improvements there.

**License**: MIT (or match the license of the repo you're porting from)
