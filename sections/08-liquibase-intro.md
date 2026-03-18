---
layout: center
---
# Liquibase Introduction

## Core Concepts

---

# The Central Idea

Liquibase manages your database schema with a **changelog** — a file that lists every change ever made to the schema, in order.

<div class="flex flex-col items-center gap-0 mt-4 select-none">

  <div class="w-64 text-center px-3 py-2 rounded-lg border-2 border-blue-400 bg-blue-50 dark:bg-blue-900/30 font-mono text-sm font-semibold text-blue-800 dark:text-blue-300">
    Your Java Code
    <div class="text-xs font-normal text-blue-600 dark:text-blue-400">@Entity classes</div>
  </div>

  <div class="w-0.5 h-4 bg-slate-400"></div>

  <div class="w-64 text-center px-3 py-2 rounded-lg border-2 border-green-400 bg-green-50 dark:bg-green-900/30 font-mono text-sm font-semibold text-green-800 dark:text-green-300">
    Liquibase Changelog
    <div class="text-xs font-normal text-green-600 dark:text-green-400">versioned schema changes</div>
  </div>

  <div class="w-0.5 h-4 bg-slate-400"></div>

  <div class="w-64 text-center px-3 py-2 rounded-lg border-2 border-amber-400 bg-amber-50 dark:bg-amber-900/30 font-mono text-sm font-semibold text-amber-800 dark:text-amber-300">
    DATABASECHANGELOG table
    <div class="text-xs font-normal text-amber-600 dark:text-amber-400">tracks what has run</div>
  </div>

  <div class="w-0.5 h-4 bg-slate-400"></div>

  <div class="w-64 text-center px-3 py-2 rounded-lg border-2 border-indigo-400 bg-indigo-50 dark:bg-indigo-900/30 font-mono text-sm font-semibold text-indigo-800 dark:text-indigo-300">
    PostgreSQL
    <div class="text-xs font-normal text-indigo-600 dark:text-indigo-400">your actual tables</div>
  </div>

</div>

---

# Key Term: Changeset

A **changeset** is the smallest unit in Liquibase. Each changeset describes one schema operation.

```yaml
- changeSet:
    id: 001-create-projects-table
    author: sunrise-team
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

<v-click>

Every changeset has:
- `id` — unique within the changelog
- `author` — who wrote it
- `changes` — the actual operations

</v-click>

---

# Key Term: Changelog

A **changelog** is a file that contains a list of changesets. It can include other changelogs too.

You typically have a **master changelog** that imports section-specific changelogs:

```yaml
# db/changelog/db.changelog-master.yaml
databaseChangeLog:
  - include:
      file: db/changelog/changes/001-create-projects-table.yaml
  - include:
      file: db/changelog/changes/002-create-users-table.yaml
  - include:
      file: db/changelog/changes/003-create-roles-table.yaml
  - include:
      file: db/changelog/changes/004-create-tasks-table.yaml
  - include:
      file: db/changelog/changes/005-create-user-role-table.yaml
```

<v-click>

The master changelog is the single entry point. Liquibase starts here and follows the includes in order.

</v-click>

---

# The DATABASECHANGELOG Table

When Liquibase runs for the first time, it creates a tracking table in your database:

```sql
SELECT id, author, filename, dateexecuted, md5sum
FROM DATABASECHANGELOG
ORDER BY dateexecuted;
```

| id | author | filename | dateexecuted |
|----|--------|----------|-------------|
| 001-create-projects-table | sunrise-team | changes/001-... | 2026-03-17 |
| 002-create-users-table | sunrise-team | changes/002-... | 2026-03-17 |
| 003-create-roles-table | sunrise-team | changes/003-... | 2026-03-17 |

<v-click>

On every startup, Liquibase checks this table. Changesets already in the table are **skipped**. New ones are **executed in order**.

</v-click>

---

# The Startup Flow

Every time your Spring Boot app starts:

<v-clicks>

1. **Liquibase runs before** your application code executes
2. It reads `db.changelog-master.yaml`
3. For each changeset listed: check if it's in `DATABASECHANGELOG`
4. If not found → **execute it** and record it in the table
5. If found → **skip it** (already applied)
6. Once all changelogs are processed → Spring Boot continues startup

</v-clicks>

<v-click>

This means migrations are **automatic on deploy** — no manual SQL to run, no forgetting to update staging.

</v-click>

---

# Supported Changelog Formats

Liquibase supports four formats — the same changeset in each:

<div class="grid grid-cols-2 gap-3 mt-3 text-xs">

<div>

**YAML**
```yaml
- changeSet:
    id: 001-create-projects
    author: team
    changes:
      - createTable:
          tableName: projects
```

</div>

<div>

**XML**
```xml
<changeSet id="001-create-projects"
           author="team">
    <createTable tableName="projects"/>
</changeSet>
```

</div>

<div>

**JSON**
```json
{ "changeSet": {
    "id": "001-create-projects",
    "author": "team",
    "changes": [{ "createTable": {...} }]
}}
```

</div>

<div>

**SQL** ← we'll use this
```sql
--changeset team:001-create-projects
CREATE TABLE projects (...);

--rollback DROP TABLE projects;
```

</div>

</div>

---

# Why SQL?

<v-clicks>

**You already know it** — no new syntax to learn on top of Liquibase

**Readable** — see exactly what runs in the database

**Copy-paste friendly** — test in psql first, then paste into the changeset

**No indentation traps** — YAML is strict about spaces; SQL isn't

</v-clicks>

<v-click>

The master changelog (the index file) stays YAML — it's just a list of `include:` entries, no SQL needed there.

</v-click>

---

# SQL Changeset Anatomy

Every `.sql` changelog file follows this pattern:

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

<v-clicks>

`--liquibase formatted sql` — required header at the top of the file

`--changeset author:id` — marks the start of a new changeset

`--rollback` — the SQL to undo this changeset (one statement per line)

</v-clicks>

<!--
The --rollback line is optional for reversible DDL (Liquibase can auto-generate DROP TABLE etc.), but explicit is always better.
-->
