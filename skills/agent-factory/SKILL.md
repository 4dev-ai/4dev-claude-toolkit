---
name: agent-factory
description: "Generate project-specific developer and triage agents from proven workflow templates. Use when the user says 'create agents for this project', 'set up developer/triage agents', 'generate agents', 'bootstrap agents for this repo', or wants Claude Code agents that handle ticket-based development workflows and GitHub project board triage. Also use when the user wants to update or regenerate existing agents for a project. This skill should trigger even when the user casually mentions wanting automated development workflows, sprint planning, or agent-based ticket management for any repository."
version: 0.1.0
---

# Agent Factory

Generate project-specific **developer** and **triage** agents from battle-tested templates. These agents enable a fully automated development workflow: the triage agent grooms the project board, the developer agent picks tickets, plans in worktrees, opens draft PRs for approval, then implements after human sign-off.

## What Gets Generated

| Agent | Role | Model | Color |
|-------|------|-------|-------|
| `{slug}-developer` | Senior developer — picks tickets, plans in worktrees, implements after approval | opus | red |
| `{slug}-triage` | PO/Scrum Master — triages board, categorizes, sizes, analyzes dependencies | sonnet | orange |

**Output locations:**
- Agent files: `<project>/.claude/agents/{slug}-developer.md` and `{slug}-triage.md`
- Memory dirs: `<project>/.claude/agent-memory/{slug}-developer/` and `{slug}-triage/`

## Board Lifecycle

The agents operate on a GitHub Projects board with this column flow:

```
Todo → Ready-For-Dev → In Progress → Ready-For-Review → Cherry-Picked → Done
```

With labels for the approval gate:
- `approval-pending` — developer agent submitted a plan, waiting for human review
- `approved` — human approved the plan, developer agent can implement
- Priority labels: `p0-critical`, `p1-important`, `p2-nice-to-have`

### How the Agents Interact

1. **Triage agent** moves issues from Todo → Ready-For-Dev (when deps resolved, well-scoped)
2. **Developer agent** picks Ready-For-Dev tickets → creates worktree → plans → Draft PR with `approval-pending`
3. **Human** reviews plan → swaps label to `approved`
4. **Developer agent** (next loop pass) picks `approved` tickets → implements → moves to Ready-For-Review
5. **Human** reviews implementation → cherry-picks commits to main → moves to Cherry-Picked → Done

---

## Generation Workflow

Follow these steps in order when generating agents.

### Step 1: Determine Project Root

```bash
git rev-parse --show-toplevel
```

Confirm the detected root with the user. All generated paths will be relative to this root.

### Step 2: Scout the Codebase

Read these files in parallel to auto-detect project metadata:

**Project identity:**
- `CLAUDE.md`, `AGENTS.md` — existing conventions and guidelines
- `README.md` — project description

**Tech stack detection:**
- `package.json` — name, description, scripts, dependencies
- `Cargo.toml` — Rust project name and dependencies
- `pyproject.toml` / `setup.py` — Python project info
- `go.mod` — Go project info

**Package manager detection** (check for lock files, first match wins):
1. `pnpm-lock.yaml` → pnpm
2. `bun.lockb` → bun
3. `yarn.lock` → yarn
4. `package-lock.json` → npm
5. `Cargo.lock` → cargo
6. `go.sum` → go modules
7. `poetry.lock` / `Pipfile.lock` → poetry/pip

**Validation command detection:**
1. Check `package.json` scripts for: `check:all`, `check`, `test`, `lint`, `validate`, `ci`
2. Check for `Makefile` with `check`, `test`, `lint` targets
3. If Rust: `cargo test && cargo clippy`
4. Fall back to asking user

**Architecture discovery:**
- List top-level dirs in `src/`, `lib/`, `app/`, `packages/`
- For Rust: parse `src/*.rs` file names as modules
- For React/Node: identify key dirs under `src/` (components, hooks, store, api, etc.)
- For monorepos: identify packages/workspaces

**GitHub details:**
```bash
gh repo view --json owner,name,description
gh project list --owner <owner> --format json
```

### Step 3: Build Project Context

Compile discovered info into this structure:

| Field | Source | Required? |
|-------|--------|-----------|
| `project_name` | package.json name, Cargo.toml name, or dir name | Yes |
| `project_slug` | kebab-case of project_name | Yes |
| `project_description` | package.json description, repo description, or ask | Yes |
| `tech_stack` | detected languages, frameworks, key libraries | Yes |
| `package_manager` | lock file detection | Yes |
| `validation_command` | scripts detection or ask | Yes |
| `architecture` | key modules/dirs discovered | Yes |
| `github_owner` | `gh repo view` | Yes |
| `github_repo` | `gh repo view` | Yes |
| `github_project_number` | `gh project list` or ask | Yes |
| `area_categories` | inferred from dir structure | Yes |
| `project_root` | absolute path from Step 1 | Yes |
| `test_counts` | discoverable from config/scripts | No |

### Step 4: Present Discovery to User

Show the gathered context as a structured summary. For each field:
- **Detected confidently** → show the value
- **Guessed** → show with a "?" marker, ask to confirm
- **Not found** → ask the user to provide

Area categories: auto-apply if the directory structure clearly maps to categories (e.g., `src/auth/` → auth, `src/api/` → api). If ambiguous, present inferred categories and ask for confirmation.

### Step 5: Ensure Project Board & Labels

Check if a GitHub project board exists:
```bash
gh project list --owner <OWNER> --format json
```

**If no board exists**, create one:
```bash
gh project create --owner <OWNER> --title "<PROJECT_NAME>"
```

Then set up columns via GraphQL: `Todo`, `Ready-For-Dev`, `In Progress`, `Ready-For-Review`, `Cherry-Picked`, `Done`.

**Ensure labels exist** on the repository:
```bash
gh label create "approval-pending" --description "Plan submitted, awaiting review" --color "fbca04" --repo <OWNER>/<REPO> 2>/dev/null
gh label create "approved" --description "Plan approved, ready to implement" --color "0e8a16" --repo <OWNER>/<REPO> 2>/dev/null
gh label create "p0-critical" --description "Critical priority" --color "b60205" --repo <OWNER>/<REPO> 2>/dev/null
gh label create "p1-important" --description "High priority" --color "d93f0b" --repo <OWNER>/<REPO> 2>/dev/null
gh label create "p2-nice-to-have" --description "Medium priority" --color "c5def5" --repo <OWNER>/<REPO> 2>/dev/null
```

Labels that already exist will error silently — that's fine.

### Step 6: Ask Which Agents to Generate

Default: both developer and triage. The user can choose:
- Both (default)
- Developer only
- Triage only
- Other types if extensible templates exist in `references/`

### Step 7: Read Reference Templates

Read the relevant template files from this skill's `references/` directory:
- `references/developer-template.md` — for developer agent
- `references/triage-template.md` — for triage agent
- `references/memory-block.md` — shared memory section

Each template has two parts:
1. **Template spec** — describes each section, what's variable vs static, with `<!-- VARIABLE -->` and `<!-- STATIC -->` markers
2. **Full example** — a complete working agent (Renotes) showing exact output format and tone

### Step 8: Generate Agent Files

For each agent type:

1. **Check for existing files** at `<project_root>/.claude/agents/<slug>-<type>.md`. If found, warn and ask before overwriting.

2. **Generate the agent** by:
   - Reading the template spec to understand the structure
   - Reading the full example to match tone and format
   - Filling in project-specific sections using the gathered context
   - Copying static sections with only path/PM/validation substitutions
   - Appending the memory block from `references/memory-block.md` with `{{AGENT_NAME}}` and `{{PROJECT_ROOT}}` replaced

3. **Write the file** to `<project_root>/.claude/agents/<slug>-<type>.md`

Key generation rules:
- The **workflow steps** are the core value — preserve them faithfully, only substituting paths and project details
- The **area categories** in the triage agent must reflect this specific project's architecture
- The **description** in frontmatter must include triggering examples specific to this project's domain
- All `gh` commands must use the correct owner, repo, and project number
- All file paths must use the correct project root
- The validation command must match what was detected

### Step 9: Create Memory Directories

```bash
mkdir -p <project_root>/.claude/agent-memory/<slug>-developer
mkdir -p <project_root>/.claude/agent-memory/<slug>-triage
mkdir -p <project_root>/.claude/agent-locks
```

### Step 10: Report

Show the user what was created:
- List of generated files with full paths
- Summary of detected project context used
- Reminder that agents can be invoked via the Agent tool or by name
- Suggestion to run `/loop` with the triage agent for continuous board grooming

---

## Extensibility

New agent types can be added by creating a `references/<type>-template.md` file following the same pattern:
1. Template spec with variable/static markers
2. Full working example
3. Reference to memory-block.md for the shared memory section

The generation workflow in Steps 7-8 will automatically pick up new templates when the user selects them in Step 6.

---

## Customization Options

- **Model override**: user can specify different models (e.g., sonnet for developer to save cost)
- **Custom name prefix**: use a different slug than the auto-detected project name
- **Skip board setup**: if the user already has a board configured differently
