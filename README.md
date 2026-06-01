# Agent Workflow — Multi-Agent Role Workflow Skill

A portable, fully local **Claude Code workflow**. Split a project into multiple **Agent roles** (main / planner / executor / reviewer + your own), each with its own responsibilities, memory, activity traces, and knowledge capture. Ideal for one person who wants to drive a complex project as a disciplined "virtual team".

> Pure local files — no server / SSH required.

> **Two depths of use:**
> - Just want the workflow → see "Install (2 steps)" below; drop the skill into an existing project.
> - Want to set up a clean terminal environment from scratch (Starship + WSL + Zellij + Claude Code + this skill) → see **[SETUP.md](SETUP.md)**.

---

## What this workflow gives you

- **Role division**: start each message with "you are executor / planner / …", and Claude switches to that role's responsibilities and memory.
- **Persistent memory**: each role has its own `MEMORY.md`; the next conversation auto-resumes from last progress.
- **Situational awareness**: a global `activity_log.md` lets every role know what the others did recently.
- **Blockers protocol**: register what's stuck, with a clear owner and how to patch it.
- **Knowledge capture (Note)**: archive important findings in a standard format so they're never lost.
- **Exec-Review auto loop**: executor ↔ reviewer auto-iterate bug fixes for up to 5 rounds.
- **Task ledger (todos)**: only planner can mark ✅, so "done" really means verified.

---

## Install (2 steps)

1. Copy the `.claude/skills/agent-workflow/` folder from this repo into your **target project root** (alongside your code).
   ```
   your-project/
   └── .claude/skills/agent-workflow/   ← copy this in
   ```
2. Open Claude Code in that project and run:
   ```
   /agent-workflow init
   ```
   It will automatically:
   - Create `.workflow/` in the project root (shared state + per-role dirs)
   - Install the workflow protocol into the project's `CLAUDE.md` (if a CLAUDE.md already exists, it writes to `.workflow/WORKFLOW.md` and tells you how to include it — never overwriting your file)
   - Ask you for a one-line project goal

After that, the workflow is "always on".

---

## Daily Usage

```
you are planner, break the goal into phases and todos
you are executor, implement phase 1 step 1
you are reviewer, test executor's recent implementation
you are executor ask, which caching strategy does this function currently use?   # ask = lightweight Q&A, no trace
exec review loop fix the concurrency bug in the login module                      # auto loop
you are reviewer note an unlocked connection pool intermittently 500s under load  # capture knowledge
planner verify phase 1 step 1                                                     # mark ✅
```

## Roles & Modes at a Glance

| Built-in role | Responsibility |
|---------------|----------------|
| `main` | Coordinate, delegate, track |
| `planner` | Maintain roadmap/todos, sole ✅ authority |
| `executor` | Implement, produce (Note Producer) |
| `reviewer` | Test, review (Note Producer) |

Add a custom role: `/agent-workflow add-agent <name>` (e.g. `designer` / `data_analysis` / `writing`).

| Mode | Trigger |
|------|---------|
| Standard execution | `you are <agent>` |
| Ask (lightweight Q&A) | `<agent> ask …` |
| Exec-Review loop | `... loop ...` |
| Note (knowledge capture) | `<agent> note …` |
| blockers register/resolve | confirm via the proposal format mid-conversation |

Full protocol: `.claude/skills/agent-workflow/templates/CLAUDE.md`.

---

## Directory Layout (after install)

```
your-project/
├── CLAUDE.md                    # workflow protocol (installed by init)
├── .claude/skills/agent-workflow/   # this skill
└── .workflow/
    ├── shared/
    │   ├── project_brief.md     # goal
    │   ├── roadmap.md           # phase framework
    │   ├── todos.md             # concrete tasks
    │   ├── blockers.md          # blockers
    │   └── activity_log.md      # global log
    └── agents/<role>/
        ├── ROLE.md              # role contract
        ├── MEMORY.md            # role memory
        └── notes/               # knowledge capture
```

---

## Customization Tips

- **Change the role set**: delete roles you don't use, or `add-agent` to add new ones; update their responsibilities in `roadmap.md` too.
- **Change trigger words**: the `ask` / `loop` / `note` keywords in the protocol can be changed to words you prefer in `CLAUDE.md`.
- **Version control**: this skill ships no git-sync logic — use your own git flow; consider tracking `.workflow/` in version control for team / multi-machine sync.

---

## What's in This Package

| Path | Purpose |
|------|---------|
| `README.md` | This file — skill features and usage |
| `SETUP.md` | From-scratch terminal environment guide (Starship + WSL + Zellij + Claude Code) |
| `.claude/skills/agent-workflow/` | The skill itself (protocol + templates) |
| `dotfiles/starship.toml` | Starship prompt config (Pastel Powerline) |
| `dotfiles/zellij/` | Zellij multi-pane config and layout |
