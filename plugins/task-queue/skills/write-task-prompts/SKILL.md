---
name: write-task-prompts
description: Break a large piece of work into numbered, self-contained task prompt files with a TASK-MANIFEST.md index, following the task-queue-system PROMPT-SPEC format. Use when asked to split work across many parallel agent sessions.
---

# Write task prompts and the manifest

The task-queue-system pattern: one task per markdown file, a numbered manifest, and each agent session told only "run task N". Every task must be completable by one agent in one session.

## File layout

`prompts/<category>/<NN>-<slug>.md`, where `<category>` is a kebab-case group, `<NN>` a zero-padded sequence number within it and `<slug>` a kebab-case summary, for example `prompts/backend/02-database-schema.md`. If the repo already has a `PROMPT-SPEC.md`, read it first and follow it over this summary.

## Required sections, in order

1. `# Prompt NN: Title`
2. `## Context`: working directory, what already exists and what the task depends on, the problem it solves.
3. `## Rules`: scope limits (work only in one directory, touch nothing outside it), branch rules, language strictness, no stubs or placeholder code, run scoped verification first.
4. `## File Ownership`: exact paths under Creates and Modifies, and optionally Does NOT touch.
5. `## Task`: numbered, concrete, verifiable deliverables with signatures or structure shown, not full implementations.
6. `## Verification`: exact commands that prove the task is done, scoped to the task directory first.
7. `## Success Criteria`: bullets describing what done looks like.

## Writing rules

- Use full paths from the repo root and define the interfaces or types to implement.
- One task per file. Split anything over about five files or 1000 lines into ordered tasks and say in each which must finish first.
- Pick one approach instead of leaving options open, and always end with verification commands.

## Build the manifest

`TASK-MANIFEST.md` is a table of number, category and file. Regenerate it from the prompt files:

```bash
echo '# Task Manifest

> Open a chat, type `@task-runner 42` (any number below). Agent reads the file and executes it.

| # | Category | File |
|---|----------|------|' > TASK-MANIFEST.md

find prompts/ -name '*.md' ! -name 'README.md' ! -path '*/archived/*' | sort | \
  awk 'BEGIN{n=1} {split($0, a, "/"); print "|", n++, "|", a[2], "|", $0, "|"}' >> TASK-MANIFEST.md
```

When adding tasks later, append rows with the next free numbers rather than renumbering, so numbers already handed to running sessions stay valid. Check that every path in the manifest exists. Commit only if the user asks.

## Handing out work

Give each session a unique number, one line each, such as `@task-runner 1` through `@task-runner 50` in VS Code Copilot Chat (the repo's `.github/agents/task-runner.agent.md` resolves the number), or the plain instruction "read TASK-MANIFEST.md, run task 7" in any other agent.
