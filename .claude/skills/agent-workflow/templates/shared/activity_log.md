# Global Activity Log (activity_log.md)

> Every agent appends one line in Step 4 (append, not overwrite). Seventh-priority read in Step 1.
> Keep the most recent 20, FIFO-drop the oldest. Header is fixed at 7 lines; the trim command depends on it, do not change the first 7 lines.

| Time | Agent | Summary |
|------|-------|---------|
