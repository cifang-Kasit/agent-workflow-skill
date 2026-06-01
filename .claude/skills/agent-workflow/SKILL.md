---
name: agent-workflow
description: Set up and run a multi-agent role workflow in a local project. Splits a project into Agent roles (main / planner / executor / reviewer + your own), each with its own responsibilities, memory (MEMORY), activity log (activity_log), blockers, and knowledge notes. Supports Ask mode, Exec-Review auto loop, Note knowledge capture, and a blockers protocol. Use this skill when the user wants to initialize the workflow, add an agent role, or asks how the workflow is used. Pure local files, no SSH.
---

# Agent Workflow Skill

A portable, fully local **multi-agent role workflow** for Claude Code. Turn a solo project into a "virtual team": different roles divide the work, each keeps its own memory, leaves traces for the others, and captures knowledge. All state lives in local Markdown files inside the project — no server needed.

## Subcommands

Decide what to do based on the user-supplied arguments (`$ARGUMENTS`):

### `init` — initialize the workflow (first use)

Scaffold the workflow in the **current project root**:

1. Confirm the project root (current working directory). Set `ROOT` = absolute path of the project root.
2. Create directories and state files:
   ```bash
   ROOT="$(pwd)"
   mkdir -p "$ROOT/.workflow/shared"
   mkdir -p "$ROOT/.workflow/agents"/{main,planner,executor,reviewer}
   ```
3. Copy this skill's `templates/` contents into place:
   - `templates/CLAUDE.md` → `<ROOT>/CLAUDE.md`
     **If `<ROOT>/CLAUDE.md` already exists**: do not overwrite! Instead write the protocol to `<ROOT>/.workflow/WORKFLOW.md`, and tell the user to add a line `@.workflow/WORKFLOW.md` at the top of their CLAUDE.md to include it, or merge manually.
   - `templates/shared/*.md` → `<ROOT>/.workflow/shared/`
   - `templates/agents/<role>/ROLE.md` → `<ROOT>/.workflow/agents/<role>/ROLE.md` (one each for main/planner/executor/reviewer)
   - Create an empty `MEMORY.md` placeholder and a `notes/` dir in each agent dir
4. Have the user fill the placeholders: ask for the project's **one-line goal** and write it to the top of `shared/project_brief.md`.
5. Verify: list the `.workflow/` tree and confirm all files are present.
6. Tell the user the next steps:
   - Edit `.workflow/shared/project_brief.md` (goal), `roadmap.md` (phase framework), `todos.md` (tasks)
   - Then start each message with "you are <agent>" to trigger the protocol
   - Add new roles with `/agent-workflow add-agent <name>`

### `add-agent <name>` — add a custom role

1. `mkdir -p "<ROOT>/.workflow/agents/<name>/notes"`
2. Copy `templates/ROLE.template.md` → `<ROOT>/.workflow/agents/<name>/ROLE.md`, replacing the placeholder `<AGENT_NAME>` with `<name>`
3. Create an empty `MEMORY.md`
4. Ask the user for the role's **one-line responsibility** and **whether it is a Note Producer**; fill them into ROLE.md
5. Remind: if the role participates in phase work, have planner register its responsibility in `roadmap.md`

### `status` — show current workflow state

Read and summarize: current phase (todos.md), each agent's last activity (activity_log.md), unresolved blockers, and a summary of each agent's MEMORY. Read-only, changes nothing.

### no args / other — explain usage

Briefly explain what this workflow is, which roles and modes exist, and how to `init`.

## Important Notes

- This skill only handles **scaffolding and structure management**. The day-to-day Step 1-5 execution protocol is **auto-applied** once `init` installs it into the project's `CLAUDE.md` (or `.workflow/WORKFLOW.md`); you don't need to call this skill each time.
- The full protocol is in `templates/CLAUDE.md` — the authoritative definition of this workflow.
- Everything is local; never introduce SSH / remote dependencies.
