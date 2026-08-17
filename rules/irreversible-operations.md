# Irreversible operations — mandatory protocol (NO EXCEPTIONS)

> Read this WHOLE page before any operation that is hard or impossible to reverse: dropping
> tables/volumes, migrating data stores, deleting files, schema changes on prod, unpublishing a release.
> Skipping any step is not allowed regardless of time pressure or apparent simplicity.

**When a fresh backup is required before proceeding:**
- Manual DDL outside the migration runner (direct SQL `DROP`, `ALTER`, `UPDATE`/`DELETE` on large tables)
- Migrations that modify or delete existing data (not just add columns/indexes/tables)
- Any data store migration (moving volumes, changing the connection target)

**When the routine periodic backup is sufficient:**
- Routine migrations through the deploy script that only add columns, indexes, or tables
  (`IF NOT EXISTS`)

## Step 1 — Write the plan first, execute second

Write out every step of the plan explicitly, including the rollback action for each step if it fails.
Show the plan to the user before executing anything. Do not start executing while planning.

## Step 2 — Scan the rules for relevant warnings

Search the project docs tree for warnings related to the operation — e.g. "TABLE vs MATVIEW", "never
`docker volume rm`", "migration". Every warning found must become an explicit numbered step in the plan
— not background knowledge.

## Step 3 — Verify the backup before making anything irreversible

A backup that has not been verified is not a backup. Before any irreversible step: verify the backup is
restorable (list its contents, check row counts, check the schema). Only then proceed.

## Step 4 — Choose the simple reliable path, not the optimised one

Do not optimise for speed or elegance at the cost of reliability. A 20-minute downtime with a proven
procedure beats a 2-minute downtime with a clever one that has unknown failure modes.

## Step 5 — State what cannot be undone

Before executing an irreversible step, explicitly say: *"This step cannot be undone. Backup verified.
Proceeding."* This is a forcing function, not a formality.

## Step 6 — Before deleting anything in production, say whose it is and what it does

When someone asks to drop a prod object — **even one described as unused** — the confirmation must first
report:

1. **who created it and when** — `git log --diff-filter=A` on the migration that added it, plus the
   migration's own header comment;
2. **what it is for**, in one sentence, from that header or the docs;
3. **what still reads it today** — grep the codebase, and for tables/views check the DB's own access
   statistics and any dependent views;
4. **size, and whether the data is derived or unique** — a materialized view is rebuildable, a base
   table is not.

Then ask. The point is that "unused" is a belief, and the person asking may be relying on a colleague's
memory rather than on a check. Where more than one person works in a repo, an object one of them wrote
may be scheduled for a cleanup the other doesn't know about. If the object turns out to be someone
else's pending work, say so and let them do it on their own schedule rather than tidying it away for
them.

## Why this protocol exists

A database volume migration once used an rsync-based approach to minimise downtime. The rsync of a live
cluster produced inconsistent data files. The database re-initialised on startup, wiping all tables.
Recovery took 2+ hours. A related table-vs-materialized-view issue was not caught because no plan was
written and the docs were not scanned against the task. A dump/restore approach with a written plan
would have taken 20 minutes and caught all edge cases.

## Related

- [Git workflow](git-workflow.md) — how changes normally reach production
- [Engineering standards](engineering-standards.md) — risks→tests→code
