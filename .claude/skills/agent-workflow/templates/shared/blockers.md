# Blocker List (blockers.md)

> Event-driven: an agent appends when it discovers "can't solve this now"; the owner deletes it once resolved.
> Fifth-priority read in Step 1; in Step 2 this agent must report blockers whose owner = itself.
> IDs auto-increment 3 digits and are never reused after deletion.

<!-- The block below is the example format; when there are no blockers this file is empty except for this note -->

<!--
### #001 [high]
- **Time**: 2026-01-01 12:00
- **Reporter**: reviewer
- **Owner**: executor
- **Related todos step**: `phase 1 / step 2 / test login module`
- **Problem**: Login endpoint intermittently 500s under concurrency; suspected unlocked connection pool in executor's implementation
- **Steps to add**: executor adds locking to the connection pool and a regression test
-->
