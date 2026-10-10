# Task Queue System — Portable Installation Guide 

<!-- three.ws:badges -->
[![GitHub stars](https://img.shields.io/github/stars/nirholas/task-queue-system?style=flat&logo=github)](https://github.com/nirholas/task-queue-system/stargazers) [![Last commit](https://img.shields.io/github/last-commit/nirholas/task-queue-system?style=flat)](https://github.com/nirholas/task-queue-system/commits) [![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen?style=flat)](https://github.com/nirholas/task-queue-system/pulls) [![AI agent friendly](https://img.shields.io/badge/AI%20agents-AGENTS.md%20%2B%20llms.txt-6d5dfc?style=flat)](https://github.com/nirholas/task-queue-system/blob/HEAD/AGENTS.md)
<!-- /three.ws:badges -->


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

> **Note:** In this standalone repo the documentation file is `README.md` — it is
> the same content that lives in `swarms` as `TASK-QUEUE-SYSTEM.md`. If you are
> copying from a clone of this repo rather than from `swarms`, substitute
> `README.md` wherever the instructions below say `TASK-QUEUE-SYSTEM.md`.

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

### Dispatch 100 Chats Fast

Use this exact pattern for high-throughput dispatch:

1. Open as many chats as you want (for example 100).
2. In each chat, paste exactly one line: `@task-runner <number>`.
3. Use unique numbers per chat to avoid duplicate work.
4. Do not paste the whole prompt content manually; the agent resolves the number from `TASK-MANIFEST.md` and reads the task file itself.

Example batch:

```text
@task-runner 1
@task-runner 2
@task-runner 3
...
@task-runner 100
```

Expected behavior per chat:
- Resolve number -> file path from `TASK-MANIFEST.md`
- Read full task `.md`
- Implement all required work
- Run scoped verification
- Report completion (and commit/push only if requested)

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

## Example: Porting to Another Repo

```bash
cd /workspaces/target-repo

# Copy core files
mkdir -p .github/agents
cp /workspaces/swarms/.github/agents/*.md .github/agents/
cp /workspaces/swarms/PROMPT-SPEC.md .
cp /workspaces/swarms/TASK-QUEUE-SYSTEM.md .

# You might already have prompts in this repo - generate manifest
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

**License**: See [LICENSE](LICENSE) — Copyright 2026 nirholas, all rights reserved.

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=nirholas/task-queue-system&type=Date)](https://www.star-history.com/#nirholas/task-queue-system&Date)

<!-- three.ws:growth -->
## Support the project

If task-queue-system saves you time, **[star it on GitHub](https://github.com/nirholas/task-queue-system)**. Stars are how other developers and AI agents find the repositories worth trusting, and they cost you one click.

Know someone who would use it? [Post on X](https://twitter.com/intent/tweet?text=task-queue-system%3A%20Portable%20guide%20for%20splitting%20work%20into%20numbered%20task%20files%20and%20dispatching%20them%20across%20many&url=https%3A%2F%2Fgithub.com%2Fnirholas%2Ftask-queue-system) · [Share on Bluesky](https://bsky.app/intent/compose?text=task-queue-system%3A%20Portable%20guide%20for%20splitting%20work%20into%20numbered%20task%20files%20and%20dispatching%20them%20across%20many%20https%3A%2F%2Fgithub.com%2Fnirholas%2Ftask-queue-system) · [Share on LinkedIn](https://www.linkedin.com/sharing/share-offsite/?url=https%3A%2F%2Fgithub.com%2Fnirholas%2Ftask-queue-system) · [Submit to Hacker News](https://news.ycombinator.com/submitlink?u=https%3A%2F%2Fgithub.com%2Fnirholas%2Ftask-queue-system&t=task-queue-system%3A%20Portable%20guide%20for%20splitting%20work%20into%20numbered%20task%20files%20and%20dispatching%20them%20across%20many) · [Share on Reddit](https://www.reddit.com/submit?url=https%3A%2F%2Fgithub.com%2Fnirholas%2Ftask-queue-system&title=task-queue-system%3A%20Portable%20guide%20for%20splitting%20work%20into%20numbered%20task%20files%20and%20dispatching%20them%20across%20many)

## Built for AI agents too

Coding agents and LLM tooling can read this repo directly: [AGENTS.md](./AGENTS.md), [llms.txt](./llms.txt), [llms-full.txt](./llms-full.txt). Point an agent at `https://github.com/nirholas/task-queue-system` and it has the context it needs.

## More from the same author

- [All repositories by nirholas](https://github.com/nirholas/nirholas#readme): the full catalog, grouped by topic
- [three.ws](https://three.ws): the platform for 3D AI agents with Solana wallets, a skill marketplace and x402 payments
- Questions or ideas: [open an issue](https://github.com/nirholas/task-queue-system/issues) or [start a discussion](https://github.com/nirholas/task-queue-system/discussions)

## Contributors

[![Contributors](https://contrib.rocks/image?repo=nirholas/task-queue-system)](https://github.com/nirholas/task-queue-system/graphs/contributors)

<!-- /three.ws:growth -->
