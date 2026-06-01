# <AGENT_NAME> — Role Contract (ROLE.md)

> This file defines `<AGENT_NAME>`'s responsibility boundaries and norms. Highest-priority read in Step 1.

## One-line Responsibility
<Write this role's core responsibility, e.g. "Turn planner's todos into concrete implementation outputs.">

## Note Producer?
<Yes / No>.
(Only a Note Producer may write notes in Note Mode; typically executor / reviewer / analysis roles.)

## What I Own
- <responsibility 1>
- <responsibility 2>

## What I Don't Own (hand off to whom)
- <non-responsibility 1> → hand to <some agent>
- <non-responsibility 2> → hand to <some agent>

## Where Outputs Go
- Always under `.workflow/agents/<AGENT_NAME>/`

## Working Norms
- After completing an assigned todos step, **do not self-mark ✅**; report "awaiting planner verification"
- When you hit a blocker, propose registering it per the blockers protocol
- On side-effect turns, run Step 4/5

## Note Producer Writing Rules (fill in only for Producer roles)
If this role is a Producer, write notes in 5 steps:
1. Confirm the trigger type (experiment comparison / counterintuitive / behavioral conclusion / hypothesis check) and the target phase
2. Determine the auto-increment number NN under `notes/phaseX.Y/`
3. Write per the "Note File Standard Format" in CLAUDE.md
4. Cite concrete numbers / code paths / comparisons; avoid vagueness
5. After writing, append one line to activity_log (Step 4)
