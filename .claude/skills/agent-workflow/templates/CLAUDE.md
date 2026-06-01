# Multi-Agent Workflow Protocol

> This file is a project-level operating protocol. Claude reads it automatically every conversation and **follows it strictly**.
> This protocol splits a project into multiple **Agent roles**, each with its own responsibilities, memory, and outputs.
> All files are **local files** — operate on them directly with the Read / Write / Edit tools. No SSH required.

---

## Directory Layout

```
<project-root>/
├── CLAUDE.md                      # This protocol (auto-applied)
└── .workflow/
    ├── shared/                    # Global shared state
    │   ├── project_brief.md       # Overall goal (the "why")
    │   ├── roadmap.md             # Phase-level framework (maintained by planner)
    │   ├── todos.md               # Decomposed concrete tasks (maintained by planner)
    │   ├── blockers.md            # Blocker list (event-driven)
    │   └── activity_log.md        # Global activity log (every agent appends)
    └── agents/
        └── <agent_name>/
            ├── ROLE.md            # Behavior contract for this role
            ├── MEMORY.md          # Interaction memory for this role
            ├── notes/             # Notes produced by this role (by phase)
            └── .turns_since_memory# Side-effect turn count since last MEMORY write
```

> Path variables: below, `<root>` = project root, `<agent>` = current role name.

---

## Built-in Generic Roles (customizable / extensible)

| Agent | Responsibility | Note Producer? |
|-------|----------------|----------------|
| `main` | Coordinator: task delegation, progress tracking, maintains activity_log & blockers | No |
| `planner` | Planning: maintains roadmap / todos; the only role allowed to check off todos | No |
| `executor` | Execution: implement, produce, build | **Yes** |
| `reviewer` | Review / test: verify quality, report issues | **Yes** |

> **Custom roles**: drop a `ROLE.md` into `.workflow/agents/<new-name>/` (use `/agent-workflow add-agent <name>`).
> E.g. research projects may add `research` / `data_analysis` / `writing`; product projects may add `designer` / `qa`.
> Whether a role is a Note Producer is declared in its own ROLE.md.

---

## Agent Identity Designation (mandatory rule)

> **Every user message must include an Agent identity (e.g. "you are executor").**
> **If a user message specifies no Agent identity, immediately prompt:**
> "Please first specify which Agent I should act as (main / planner / executor / reviewer / <custom>)."
> **Do not guess, do not reuse the previous identity, do not start working. Wait until the user explicitly specifies.**

---

## Invocation (mandatory)

> When the user says "you are XX agent" (without "ask"), you must complete the following steps **in order** before doing anything else. Do not skip.

### Step 1 — Read core files (descending priority, read in parallel)

| # | File | Priority | Meaning |
|---|------|----------|---------|
| 1 | `<root>/.workflow/agents/<agent>/ROLE.md` | Highest | What I **should** do (behavior contract) |
| 2 | `<root>/.workflow/shared/project_brief.md` | 2nd | Project **overall goal** (the why) |
| 3 | `<root>/.workflow/shared/roadmap.md` | 3rd | **Overall framework** of the path (phase level) |
| 4 | `<root>/.workflow/shared/todos.md` | 4th | Decomposed **concrete tasks** |
| 5 | `<root>/.workflow/shared/blockers.md` | 5th | **Unplanned patches** (what's stuck, who owns it) |
| 6 | `<root>/.workflow/agents/<agent>/MEMORY.md` | 6th | What I **have done** |
| 7 | `<root>/.workflow/shared/activity_log.md` | 7th | What **other agents** have done (situational awareness) |

> On conflict, higher priority wins.
> If MEMORY.md / blockers.md / activity_log.md don't exist, just skip — don't error.

### Step 2 — Confirm current state

- From MEMORY.md, recall what this agent did last time
- From activity_log.md, learn what other agents did recently
- From todos.md, determine the current Phase
- From roadmap.md, confirm this agent's responsibility in the current Phase
- **Check blockers.md for blockers whose owner = this agent; if any, proactively report: "Found N blockers assigned to me: #NNN …, handle them this round?"**
- Briefly report to the user: current Phase, last progress, recent cross-agent activity, relevant blockers, todos, and what you're about to do

### Step 3 — Execute per role

- Follow the responsibilities and norms in ROLE.md
- All output files go under `<root>/.workflow/agents/<agent>/`

### Step 4 — Append activity_log + bump counter (required on every side-effect turn)

> **"Side-effect"** = this turn changed files / ran commands. Pure Q&A, discussion, ask mode → **skip Step 4 and Step 5**.

One bash call does "append + trim + count":
```bash
ROOT="<root>"; AGENT="<agent>"
TS=$(date '+%Y-%m-%d %H:%M')
echo "| $TS | $AGENT | <one-line summary of this task> |" >> "$ROOT/.workflow/shared/activity_log.md"
# Trim: keep 7 header lines + most recent 20 entries
head -7 "$ROOT/.workflow/shared/activity_log.md" > /tmp/al.tmp
tail -n +8 "$ROOT/.workflow/shared/activity_log.md" | tail -20 >> /tmp/al.tmp
mv /tmp/al.tmp "$ROOT/.workflow/shared/activity_log.md"
# Bump counter
n=$(cat "$ROOT/.workflow/agents/$AGENT/.turns_since_memory" 2>/dev/null || echo 0)
n=$((n+1)); echo $n > "$ROOT/.workflow/agents/$AGENT/.turns_since_memory"
echo "COUNT=$n"
```
**Read the `COUNT=N` in the output** — N decides whether Step 5 triggers.

### Step 5 — Update Agent MEMORY.md (on any trigger below)

> **Triggers (any one → update + reset counter):**
> 1. User switches to another Agent
> 2. User explicitly says "stop / pause / continue next time"
> 3. Conversation is about to end (context near limit)
> 4. **COUNT >= 6 returned by Step 4**
> 5. A one-off large task finishes (e.g. an exec-review loop completes)
>
> This rule applies to all Agents.

Writing MEMORY **must** reset the counter at the same time:
```bash
ROOT="<root>"; AGENT="<agent>"
cat > "$ROOT/.workflow/agents/$AGENT/MEMORY.md" << 'MEMEOF'
# <Agent Name> — Interaction Log

## Most Recent Interaction
- **Time**: YYYY-MM-DD HH:MM
- **User request (summary)**: (one line)
- **Work done (summary)**: (what was actually completed)
- **Current state**: (which step, anything unfinished)
- **Next-time continuation hint**: (where to resume)

## History
(Move the previous "Most Recent Interaction" here; keep the latest 20, newest first)
MEMEOF
echo 0 > "$ROOT/.workflow/agents/$AGENT/.turns_since_memory"
```

**MEMORY.md rules:**
- Each Agent has its own MEMORY.md; "Most Recent Interaction" keeps only the newest entry, older ones move into "History"
- History keeps at most 20; drop the oldest beyond that
- Use `date` for system time
- Writing MEMORY must reset `.turns_since_memory` to 0

**activity_log.md rules:**
- A single global file; every agent appends one line in Step 4 (append, not overwrite)
- Keep the most recent 20, FIFO-drop the oldest; the 7-line header structure is required by the trim command

---

## Ask Mode (lightweight Q&A)

> User message contains "ask" (e.g. "executor ask …") → enter Ask Mode.

- **Still read** ROLE.md and MEMORY.md (for role & context), but **do not report state, do not run Step 2-3**
- **Do not update** MEMORY.md by default; **do not run Step 4/5**
- **Do not create** any persistent file; if you need a temp file to extract data, **delete it immediately** after answering
- **Only answer** the user's question; do no extra work
- If the ask discussion yields a major conclusion:
  - Matches one of the 4 hard Note triggers → ask "generate a note?" per Note Mode format
  - Other insight → ask "This finding: [summary]. Update MEMORY?"
  - The two prompts never appear at once

---

## Exec-Review Loop Mode (auto loop)

> User message contains "loop" (e.g. "exec review loop fix bug in X") → enter auto loop.
> This is the executor ↔ reviewer auto-collaboration loop (i.e. "implement → test → fix").

**Loop rules:**
1. **At most 5 rounds**, or terminate early when reviewer reports all pass
2. Each round has two phases:
   - **Phase A — executor**: modify per the task (round 1) or reviewer's issue report (later rounds); write a change note into the `reviewer/` dir
   - **Phase B — reviewer**: read the change note, write/update tests, run them, output results. All PASS → terminate; any FAIL → record issues & suggestions, go to next round
3. **After the loop, update both Agents' MEMORY.md**
4. **Brief report each round**: `Loop N/5: executor changed X, reviewer tested Y PASS / Z FAIL`
5. **Still not all passing after 5 rounds** → summarize unresolved issues, hand to user

**After the loop — reviewer generates a change summary** `reviewer/<task_id>_changelog.md`:
```markdown
# Change Summary — <task name>

## Changed Files
| File | Change type | Notes |
|------|-------------|-------|

## Backup Locations
| Original file | Backup path |
|---------------|-------------|

## Rollback
(How to restore the pre-change state)

## Test Results
- Total rounds: N
- Final result: X PASS / Y FAIL
- Test script: reviewer/test_xxx.*
```

---

## Note Mode (knowledge capture)

> Trigger: user message contains `note`, format `<agent> note <content>`.

**Default producer restriction**: only roles declared as Producer in their ROLE.md (default executor / reviewer + custom analysis roles) may write notes.
**User-sovereignty exception**: if the user issues `note` to a non-producer, warn first, then proceed:
> "Note Mode is usually written by producers like executor/reviewer. As you explicitly requested, I'll follow the producer flow this time."

**Note Mode behavior:**
- Step 1: **read all core files in full** (keep context complete)
- Step 2: **skip the state report**, go straight to writing the note
- Step 3: follow the "Note Producer writing rules" in the relevant producer's ROLE.md
- Step 4 / 5: as usual

**Auto-proposal trigger (self-check by agent in normal mode)**: when one of these 4 **hard triggers** is met, you must proactively propose at the end of your response:

| Type | Example |
|------|---------|
| Experiment/approach comparison with significant result | New method clearly beats baseline |
| Counterintuitive phenomenon | Opposite to expectation / literature |
| Reproducible behavioral conclusion | "Usage X crashes under condition Y" |
| Confirm or refute a clear hypothesis | "Hypothesis A holds / does not hold" |

**Do NOT propose** (to avoid polluting notes/): routine single-run results, ad-hoc debugging observations, facts the user already acknowledged verbally.

**Proposal format:**
```
This finding may be worth recording as a note:
- Type: experiment comparison / counterintuitive / behavioral conclusion / hypothesis check
- Description: <one or two sentences>
- Suggested phase: phase X.Y
Generate? (y/n/adjust)
```
User replies `y` or adjusts → run the 5-step flow; `n` → skip.

### Note File Standard Format

Stored at `<agent>/notes/phaseX.Y/NN.<slug>.md` (NN auto-increments 2 digits within the phase):
```markdown
# Note: <title>

**ID**: phaseX.Y/#NN
**Date**: YYYY-MM-DD
**Author**: <agent_name>
**Trigger**: user instruction / agent proposal
**Related**: <related plan / todo> (if any)

## TL;DR
2-4 sentences of core conclusion

## Background
Why this note, where the question came from

## Methodology / Scope
Experiment type → data/params/comparisons; concept type → what is and isn't covered

## Findings
Core content (tables / numbers / code paths)

## Discussion
Interpretation, limitations, surprises

## Takeaways
- Actionable conclusion 1
- Actionable conclusion 2
```

**Lifecycle:** created by the producer; in principle **not edited** (point-in-time); when outdated → write a new note and mark the old one at the top with `## Status: SUPERSEDED by phaseX.Y/#NN`; never delete.

---

## blockers.md Protocol (event-driven, not a Step)

> An agent appends when it **discovers** "can't solve this now" during execution; the owner deletes it once resolved.

**The 4 scenarios where an agent must proactively propose a blocker:**
1. **Cross-agent dependency** — this agent can't finish it; another agent is needed
2. **Missing external resource** — not solvable by code (missing data / key / dependency not installed)
3. **Task stuck by external cause** — can't proceed this turn
4. **Out-of-scope discovery** — a related issue found incidentally, outside the current task

**Do NOT propose**: small issues solvable this turn; style/refactor suggestions; next steps already in todos.

**Proposal format:**
```
This may need a blocker:
- Type: cross-agent dependency / external resource / task stuck / out-of-scope
- Description: <one or two sentences>
- Suggested owner: <executor / user / ...>
- Suggested priority: high / medium / low
Register it?
```
User replies "yes/register" → append; otherwise → skip.

**Append command (auto-increment ID):**
```bash
ROOT="<root>"; B="$ROOT/.workflow/shared/blockers.md"
NEXT_ID=$(grep -oE '#[0-9]{3}' "$B" | sort -u | tail -1 | tr -d '#')
NEXT_ID=$(printf '%03d' $((10#${NEXT_ID:-0}+1)))
TS=$(date '+%Y-%m-%d %H:%M')
cat >> "$B" << BLKEOF

### #$NEXT_ID [<high/medium/low>]
- **Time**: $TS
- **Reporter**: <agent_name>
- **Owner**: <agent_name or user>
- **Related todos step**: \`<Phase X.Y / Step N / description>\`
- **Problem**: <one or two sentences: symptom & impact>
- **Steps to add**: <what the owner should add to todos>
BLKEOF
echo "BLOCKER_ADDED=$NEXT_ID"
```

**3 mandatory steps when resolving:**
1. Delete the whole block from blockers.md
2. Append a dedicated line to activity_log: `| TS | <agent> | resolved blocker #NNN: <original steps-to-add> |`
3. Note "resolved #NNN" in your own MEMORY

**blockers.md rules:** IDs auto-increment 3 digits and are never reused after deletion; owner must be explicit (a specific agent or user); "steps to add" must be actionable; required reading in Step 1, must be reported in Step 2.

---

## todos.md Completion Reporting (cross-agent rule: never self-check ✅)

> After any executing agent completes an assigned todos step, it is **forbidden to mark ✅ in todos.md itself**.
> The sole check-off authority is **planner**, triggered by an explicit user instruction (see `planner/ROLE.md`).

**After an executing agent finishes, it must:**
1. Update its own MEMORY.md (Step 5)
2. Leave a trace in activity_log (Step 4)
3. **Report to the user**: `phase X.Y / step N done, awaiting planner verification`

**User verification flow:**
1. User instructs planner: `planner verify phase X.Y step N`
2. Planner cross-verifies from multiple sources (activity_log / MEMORY / reading the output files directly)
3. Mark ✅ / 🔄 / ☐ per result
4. Report the verification result

**Rationale**: todos is the project's ledger of facts; ✅ must mean "independently verified".

---

## Operating Norms

- All file operations are **local** — use Read / Write / Edit directly; for atomic state-file ops (counters, trimming, ID increment) use the bash snippets above
- Verify after each write: `ls -la <file>` or Read to confirm
- Put output files under the corresponding agent dir; don't scatter them into the project root
