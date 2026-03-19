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
  db/changelog/changes/001-create-projects-table.sql::001-create-projects-table::sunrise-team
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

Use `insert` changesets to populate reference or test data:

```sql
--liquibase formatted sql

--changeset sunrise-team:007-seed-roles
INSERT INTO roles (name) VALUES ('VIEWER');
INSERT INTO roles (name) VALUES ('USER');
INSERT INTO roles (name) VALUES ('ADMIN');
INSERT INTO roles (name) VALUES ('SUPER_ADMIN');

--rollback DELETE FROM roles WHERE name IN ('VIEWER','USER','ADMIN','SUPER_ADMIN');
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

<Transform :scale="0.9">

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
      file: db/changelog/changes/007-seed-roles.sql
  - include:
      file: db/changelog/changes/008-seed-dev-data.sql
```

</Transform>

</v-click>

---

# Preconditions

What if someone manually inserted `ADMIN` directly in the dev DB — then the team deploys? The seed changeset runs again and throws a **duplicate key error**.

<v-click>

Preconditions let you check first — run the changeset only if the condition is met:

```sql
--liquibase formatted sql

--changeset sunrise-team:007-seed-roles
--preconditions onFail:MARK_RAN
--precondition-sql-check expectedResult:0 SELECT COUNT(*) FROM roles WHERE name IN ('VIEWER','USER','ADMIN','SUPER_ADMIN')
INSERT INTO roles (name) VALUES ('VIEWER');
INSERT INTO roles (name) VALUES ('USER');
INSERT INTO roles (name) VALUES ('ADMIN');
INSERT INTO roles (name) VALUES ('SUPER_ADMIN');

--rollback DELETE FROM roles WHERE name IN ('VIEWER','USER','ADMIN','SUPER_ADMIN');
```

</v-click>

<v-click>

- **`onFail:MARK_RAN`** — roles already exist? Record it as done and move on silently
- **`onFail:HALT`** — fail the deployment (use for critical structural checks)
- **`onFail:SKIP`** — skip without recording (changeset will attempt again next time)

</v-click>

---

# Raw SQL Escape Hatch

When YAML operations aren't enough, use raw SQL — like inserting roles with a select-based guard:

```sql
--liquibase formatted sql

--changeset sunrise-team:008-seed-roles-raw-sql
INSERT INTO roles (name)
SELECT r.name
FROM (VALUES ('VIEWER'), ('USER'), ('ADMIN'), ('SUPER_ADMIN')) AS r(name)
WHERE NOT EXISTS (SELECT 1 FROM roles WHERE roles.name = r.name);

--rollback DELETE FROM roles WHERE name IN ('VIEWER','USER','ADMIN','SUPER_ADMIN');
```

<v-click>

Use raw SQL for:
- `INSERT ... SELECT` with conditional logic
- Data transformations
- Database-specific features not covered by Liquibase operations
- Bulk operations

</v-click>

<!--
Warn: raw SQL is less portable across databases. Fine for PostgreSQL-only projects.
-->
