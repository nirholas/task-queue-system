# Prompt Specification

> This file defines the **exact format** for task prompt `.md` files in this repo. Any agent creating prompts for other agents MUST follow this spec.

## File Extension & Location

- **Extension**: `.md` (plain markdown)
- **Location**: `prompts/<category>/<NN>-<slug>.md`
  - `<category>` = subdirectory grouping related tasks (e.g., `swarm`, `agentos-os-ui`, `pump-agent-swarm`, `agents`)
  - `<NN>` = zero-padded sequence number within the category (e.g., `01`, `02`, `25`)
  - `<slug>` = kebab-case short description (e.g., `event-bus`, `fix-build-errors`, `system-tray-quick-settings`)
- **Examples**:
  - `prompts/swarm/04-event-bus.md`
  - `prompts/agentos-os-ui/01-system-tray-quick-settings.md`
  - `prompts/pump-agent-swarm/23-error-handling-retries.md`

## Required Sections

Every prompt file MUST have these sections in this order:

```markdown
# Prompt NN: Title

## Context
Where in the repo this work happens. What exists already. What the goal is.
- Working directory (e.g., "Work in `/workspaces/swarms/packages/pump-agent-swarm/`")
- What files/modules exist that this task depends on
- What problem this solves

## Rules
- Work ONLY in `<specific directory>` — do NOT touch files outside this scope
- Stay on branch `main` — do NOT create new branches
- ALWAYS use `isBackground: true` for terminal commands and ALWAYS kill every terminal after getting output
- Commit and push as `nirholas`
- TypeScript strict mode — no `any`, no `@ts-ignore`
- No mocks, no stubs, no placeholder code, no TODOs — full real implementations only
- Connect to real APIs with real data
- Do not spawn subagents or delegate implementation
- Run scoped verification first (package/app-level checks when available)
- Do not run full `npx tsc --noEmit` unless explicitly requested by the user for that task
- Run tests after changes

## File Ownership
- **Creates**: list of files this task creates (exact paths)
- **Modifies**: list of files this task modifies (exact paths)
- **Does NOT touch**: boundaries (optional, for clarity)

## Task
Numbered list of specific deliverables. Each item should be concrete and verifiable.
1. Create/modify `path/to/file.ts` with:
   - Specific class/function/interface requirements
   - Expected behavior
   - Code snippets showing signatures or structure (not full implementations — let the agent write real code)
2. ...

## Verification
Exact commands to verify the task is complete:
```bash
# Prefer scoped checks first (adjust directory to task scope)
npm -C <task-scope-dir> run typecheck || npm -C <task-scope-dir> run type-check

# Do not run full repo typecheck unless user explicitly asks
# npx tsc --noEmit

npx vitest run           # Tests pass
npm run build            # Build succeeds
```

## Success Criteria
Bullet list of what "done" looks like:
- File X exists and exports Y
- Function Z handles edge case W
- Tests cover A, B, C
```

## Rules for Prompt Content

1. **Be specific about file paths** — always use full relative paths from repo root.
2. **Define interfaces/types** — show the agent what signatures to implement, not full code.
3. **One task per file** — each `.md` file should be completable by a single agent in a single chat session.
4. **Scope boundaries** — explicitly state what directory the agent works in and what it must NOT touch.
5. **No ambiguity** — if there are multiple ways to do something, pick one and specify it.
6. **Include dependencies** — list what files/modules the new code depends on so the agent can read them.
7. **Include verification** — always end with exact commands to prove the task is done.

## Sizing Guidelines

- **Small task** (1 file, < 200 lines): Single prompt file, 1 agent.
- **Medium task** (2-5 files, 200-1000 lines): Single prompt file, 1 agent.
- **Large task** (5+ files, 1000+ lines): Split into multiple prompt files with clear dependency order. Number them sequentially (`01-...`, `02-...`) and note in each file which tasks must complete first.

## Category Naming Conventions

| Category | Use for |
|----------|---------|
| `swarm` | pump-agent-swarm internal modules |
| `pump-agent-swarm` | pump-agent-swarm wiring, tests, deployment |
| `agents` | Backend agent route handlers, source adapters |
| `agentos-*` | AgentOS UI features (desktop, enhancements, etc.) |
| `scale` | Infrastructure scaling tasks |
| `news` | News app features |
| `platform` | Cross-cutting platform work (payments, hosting) |

When creating a new category, use kebab-case and create the directory under `prompts/`.

## After Writing Prompts

After creating prompt files, the prompt-writer agent MUST:

1. **Add entries to `TASK-MANIFEST.md`** — append new rows to the manifest table with the next available task numbers.
2. **Verify each file exists** at the path listed in the manifest.
3. **Commit and push** all new files:
   ```bash
   git config user.name "nirholas" && git config user.email "nirholas@users.noreply.github.com"
   git add prompts/ TASK-MANIFEST.md
   git commit -m "chore: add N new task prompts for <category>"
   git push
   ```
