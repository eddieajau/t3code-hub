Archive completed tasks from TODO.md into HISTORY.md.

## Steps

1. Read `TODO.md` and `HISTORY.md`.

2. Identify **fully completed tasks** — a task is fully completed when:
   - Every non-Verify checkbox is checked (`[x]`)
   - Verify checkboxes are ignored (they are always unchecked by design)
   - If a task has no checkboxes at all, it is NOT considered complete

3. If no tasks are fully complete, report that and stop.

4. For each completed task, append to `HISTORY.md` grouped by date:
   - If a `## YYYY-MM-DD` heading for today already exists, add bullet points under it.
   - Otherwise, append a new date heading first.
   ```
   ## YYYY-MM-DD

   - Task N: <title>
   - Task N: <title>
   ```
   Use today's date. Keep entries in chronological order (append at end).

5. Remove the completed task sections (heading through all content until the next `###` heading or end of file) from `TODO.md`.

6. Show the user what was archived and what remains, then ask them to confirm before committing.

7. When confirmed, commit both files with message:
   ```
   Archive completed tasks: <comma-separated task titles>
   ```
