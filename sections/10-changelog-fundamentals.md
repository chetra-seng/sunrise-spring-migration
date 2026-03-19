---
layout: center
---
# Changelog Fundamentals

## Writing Your First Changesets

---

# The createTable Operation

The most common first changeset: creating a table from scratch.

```sql
--liquibase formatted sql

--changeset sunrise-team:001-create-projects-table
CREATE TABLE projects (
    id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name       VARCHAR(255),
    created_at TIMESTAMP
);

--rollback DROP TABLE projects;
```

<v-click>

Liquibase reads the `--changeset` comment, runs the SQL below it, and records it in `DATABASECHANGELOG`. That's it.

</v-click>

---
zoom: 0.85
---

# Full Tasks Table Changeset

```sql
--liquibase formatted sql

--changeset sunrise-team:004-create-tasks-table
CREATE TABLE tasks (
    id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title      VARCHAR(100) NOT NULL,
    description TEXT,
    completed  BOOLEAN DEFAULT false,
    created_at TIMESTAMP,
    project_id BIGINT REFERENCES projects(id)
);

--rollback DROP TABLE tasks;
```

---

# The addColumn Operation

Adding a column to an existing table — this is what you'll do most often as the app evolves.

```sql
--liquibase formatted sql

--changeset sunrise-team:008-add-assignee-to-tasks
ALTER TABLE tasks ADD COLUMN assignee VARCHAR(50);

--rollback ALTER TABLE tasks DROP COLUMN assignee;
```

<v-click>

Note: Adding a nullable column works safely on existing tables. Adding a `NOT NULL` column without a default fails if there are existing rows — add a default or backfill first.

</v-click>

---

# Multiple Changesets in One File

Each `.sql` file can hold multiple changesets — one per schema change:

```sql
--liquibase formatted sql

--changeset sunrise-team:006-add-task-indexes
CREATE INDEX idx_tasks_completed  ON tasks(completed);
CREATE INDEX idx_tasks_project_id ON tasks(project_id);

--rollback DROP INDEX idx_tasks_completed;
--rollback DROP INDEX idx_tasks_project_id;

--changeset sunrise-team:007-add-assignee
ALTER TABLE tasks ADD COLUMN assignee VARCHAR(50);

--rollback ALTER TABLE tasks DROP COLUMN assignee;
```

<v-click>

Or keep one changeset per file — easier to find and review in Git. Either works.

</v-click>

---

# SQL vs YAML — Same Thing, Different Syntax

For reference, here's the same `CREATE TABLE` in SQL and YAML:

<div class="grid grid-cols-2 gap-4 mt-2 text-xs">

<div>

**SQL** (what we use)
```sql
--changeset team:001-create-projects
CREATE TABLE projects (
    id   BIGINT GENERATED ALWAYS
              AS IDENTITY PRIMARY KEY,
    name VARCHAR(255)
);
--rollback DROP TABLE projects;
```

</div>

<div>

**YAML** (what you'll see in docs)
```yaml
- changeSet:
    id: 001-create-projects
    author: team
    changes:
      - createTable:
          tableName: projects
          columns:
            - column:
                name: id
                type: BIGINT
                autoIncrement: true
                constraints:
                  primaryKey: true
```

</div>

</div>

<v-click>

Both produce identical results. SQL is more familiar and easier to test first in a database client.

</v-click>

---

# Common Changeset Operations (1/2)

Beyond `createTable` and `addColumn`, you'll frequently use:

<v-clicks>

| Operation | What it does |
|----------|-------------|
| `createTable` | Create a new table |
| `addColumn` | Add a column to existing table |
| `dropColumn` | Remove a column |
| `renameColumn` | Rename a column |
| `modifyDataType` | Change a column's type |

</v-clicks>

---

# Common Changeset Operations (2/2)

<v-clicks>

| Operation | What it does |
|----------|-------------|
| `createIndex` | Add an index |
| `addForeignKeyConstraint` | Add a foreign key |
| `insert` | Insert a row (seed data) |
| `sql` | Run raw SQL (escape hatch) |

</v-clicks>

---

# The Golden Rule

<div class="text-center mt-8">

> **Never modify a changeset that has already been applied to any database.**

</div>

<v-click>

Liquibase stores an MD5 checksum of each changeset. If you modify a changeset that's already been applied, Liquibase will detect the mismatch and **refuse to start the application**.

</v-click>

<v-click>

Instead: write a **new changeset** that makes the additional change.

This is the most important rule in Liquibase. Violating it breaks deployments.

</v-click>

<!--
Emphasize this. The temptation is to edit a changeset to fix a mistake. The correct approach is always a new changeset.
-->
