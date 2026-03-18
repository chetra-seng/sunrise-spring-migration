---
layout: center
---
# Advanced Changelogs

## Indexes, Seed Data & Preconditions

---

# The Golden Rule (Again)

Before we go further — this rule is so important it deserves its own slide:

<div class="text-center mt-8 p-6 border-2 border-red-400 rounded-lg bg-red-50 dark:bg-red-950">

## Never modify a changeset that has already been applied.

</div>

<v-click>

Liquibase stores an MD5 checksum. If you change the changeset content, the next startup fails with:

```
Caused by: liquibase.exception.ValidationFailedException:
  Validation Failed: 1 change sets check sum
  db/changelog/changes/001-create-projects-table.yaml::001-create-projects-table::sunrise-team
  was: 9:abc123... but is now: 9:def456...
```

</v-click>

<v-click>

**Always write a new changeset.** Old ones are immutable once applied.

</v-click>

---

# Adding Indexes

Indexes speed up queries. Add them in a separate changeset after creating the table:

```sql
--liquibase formatted sql

--changeset sunrise-team:006-add-task-indexes
CREATE INDEX idx_tasks_completed  ON tasks(completed);
CREATE INDEX idx_tasks_project_id ON tasks(project_id);
CREATE INDEX idx_tasks_created_at ON tasks(created_at);

--rollback DROP INDEX idx_tasks_completed;
--rollback DROP INDEX idx_tasks_project_id;
--rollback DROP INDEX idx_tasks_created_at;
```

<v-click>

Rule of thumb: add an index for every column you `WHERE` on frequently. `completed`, `project_id`, and `created_at` are used in the `/filter` endpoint's query predicates.

</v-click>

---
zoom: 0.85
---

# Seeding Initial Data

Use `insert` changesets to add reference data or dev data:

```sql
--liquibase formatted sql

--changeset sunrise-team:007-seed-initial-data
INSERT INTO projects (name, created_at)
    VALUES ('Task Management System', CURRENT_TIMESTAMP);
INSERT INTO projects (name, created_at)
    VALUES ('E-Commerce Platform', CURRENT_TIMESTAMP);
INSERT INTO tasks (title, description, completed, project_id, created_at)
    VALUES ('Design login page UI',
            'Create wireframes and implement the login form',
            true, 1, CURRENT_TIMESTAMP);

--rollback DELETE FROM tasks WHERE title = 'Design login page UI';
--rollback DELETE FROM projects WHERE name IN ('Task Management System', 'E-Commerce Platform');
```

---

# Changeset Ordering

The order of your changesets matters. Follow these rules:

<v-clicks>

1. **Create tables before inserting data** — can't insert into a table that doesn't exist
2. **Create tables before adding indexes** — indexes reference table columns
3. **Create referenced tables before foreign keys** — FK targets must exist
4. **Run seed data last** — after all structure is in place

</v-clicks>

<v-click>

```yaml
# Good ordering in master changelog:
databaseChangeLog:
  - include:
      file: db/changelog/changes/001-create-projects-table.sql
  - include:
      file: db/changelog/changes/002-create-users-table.sql
  - include:
      file: db/changelog/changes/003-create-roles-table.sql
  - include:
      file: db/changelog/changes/004-create-tasks-table.sql
  - include:
      file: db/changelog/changes/005-create-user-role-table.sql
  - include:
      file: db/changelog/changes/006-add-task-indexes.sql
  - include:
      file: db/changelog/changes/007-seed-initial-data.sql
```

</v-click>

---

# Preconditions

Preconditions run before a changeset to check if it should execute:

```sql
--liquibase formatted sql

--changeset sunrise-team:008-add-assignee-to-tasks
--preconditions onFail:SKIP
--precondition-sql-check expectedResult:0 SELECT COUNT(*) FROM information_schema.columns WHERE table_name='tasks' AND column_name='assignee'
ALTER TABLE tasks ADD COLUMN assignee VARCHAR(50);

--rollback ALTER TABLE tasks DROP COLUMN assignee;
```

<v-click>

`onFail:SKIP` — if the precondition fails, skip this changeset (instead of erroring). Useful for idempotent scripts.

Other options: `onFail:MARK_RAN` (record as run but skip), `onFail:HALT` (fail the deployment).

</v-click>

---

# Raw SQL Escape Hatch

When YAML operations aren't enough, use raw SQL:

```sql
--liquibase formatted sql

--changeset sunrise-team:009-mark-old-tasks-complete
UPDATE tasks
SET completed = true
WHERE created_at < NOW() - INTERVAL '90 days'
  AND completed = false;

--rollback UPDATE tasks SET completed = false WHERE created_at < NOW() - INTERVAL '90 days';
```

<v-click>

Use `sql:` for:
- Data transformations
- Complex `UPDATE` or `INSERT ... SELECT` statements
- Database-specific features not covered by Liquibase operations
- Bulk operations

</v-click>

<!--
Warn: raw SQL is less portable across databases. Fine for PostgreSQL-only projects.
-->
