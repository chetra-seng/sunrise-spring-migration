---
layout: center
---
# Best Practices & Environments

## Doing Migrations Right

---

# Naming Conventions

Consistent naming makes your changelog directory readable at a glance:

<v-clicks>

**Pattern:** `NNN-verb-subject.yaml`

</v-clicks>

<v-click>

```
001-create-projects-table.yaml
002-create-users-table.yaml
003-create-roles-table.yaml
004-create-tasks-table.yaml
005-create-user-role-table.yaml
006-add-task-indexes.yaml
007-seed-dev-data.yaml
008-add-assignee-to-tasks.yaml
```

</v-click>

<v-clicks>

- Numbers pad to 3 digits — alphabetical order = execution order
- Verb first: `create`, `add`, `remove`, `rename`, `update`, `seed`
- Subject is the table or column being changed
- One purpose per file

</v-clicks>

---

# One Change Per Changeset Rule

**Do this:** one logical change per changeset.

```yaml
# Good: focused changesets
- changeSet:
    id: 006-add-task-indexes
    ...
- changeSet:
    id: 008-add-assignee-to-tasks
    ...
```

<v-click>

**Not this:** multiple unrelated changes bundled together.

```yaml
# Bad: mixed concerns in one changeset
- changeSet:
    id: 006-misc-schema-changes
    changes:
      - createIndex: ...  # tasks index
      - addColumn: ...    # assignee on tasks
      - createTable: ...  # unrelated new table
```

</v-click>

<v-click>

Focused changesets roll back cleanly. Mixed changesets are a nightmare to undo.

</v-click>

---

# Changeset ID Best Practices

IDs must be unique within a changelog. Common patterns:

<v-clicks>

```yaml
# Sequential number (most common for teams)
id: 001-create-projects-table

# Date-based (useful for large teams)
id: 20260317-create-projects-table

# Ticket-based (traceability to sprint/issue)
id: TASK-42-add-assignee-column
```

</v-clicks>

<v-click>

**Pick one pattern and stick to it.** The important thing is uniqueness and readability. Sequential numbers are simplest for beginners.

</v-click>

---

# Environment Strategy

<div class="grid grid-cols-3 gap-4 mt-4">

<div class="border rounded-lg p-3 border-blue-300 bg-blue-50 dark:bg-blue-950 text-sm">

**Development**
```properties
ddl-auto=validate
liquibase.contexts=dev
```
- Seed data for testing
- Show SQL in logs
- Rebuild DB freely

</div>

<div class="border rounded-lg p-3 border-amber-300 bg-amber-50 dark:bg-amber-950 text-sm">

**Staging**
```properties
ddl-auto=validate
liquibase.contexts=staging
```
- Production-like data
- Same migrations as prod
- Full regression testing

</div>

<div class="border rounded-lg p-3 border-green-300 bg-green-50 dark:bg-green-950 text-sm">

**Production**
```properties
ddl-auto=none
liquibase.contexts=prod
```
- No seed data
- Migrations reviewed
- Rollback plan ready

</div>

</div>

<v-click>

Key principle: **the same changelog runs everywhere**. Contexts control what's different — not separate changelog files.

</v-click>

---

# CI/CD Integration

In a real deployment pipeline, Liquibase runs automatically:

<div class="mt-4 text-sm">

```
Code Push → CI Build → Tests Pass → Deploy to Staging
                                         ↓
                              Liquibase runs migrations
                              App starts with new schema
                              Integration tests run
                                         ↓
                              Deploy to Production
                                         ↓
                              Liquibase runs migrations
                              App starts (zero downtime)
```

</div>

<v-click>

You don't need to SSH into a server and run SQL scripts. Liquibase runs on startup — automatically, in every environment.

</v-click>

---

# What to Commit to Git

Always commit:

<v-clicks>

✅ All changelog files (`db/changelog/changes/*.yaml`)

✅ Master changelog (`db.changelog-master.yaml`)

✅ application.properties changes

</v-clicks>

<v-clicks>

Never commit:

❌ Liquibase lock files (if any appear locally)

❌ Generated SQL files from `generateChangeLog` (unless you're bootstrapping)

❌ Modified changesets that have already been applied to any shared DB

</v-clicks>

---

# Security: Credentials in Properties

In production, never hardcode credentials in `application.properties`:

```properties
# Bad — credentials in code
spring.datasource.password=password

# Good — credentials from environment variables
spring.datasource.url=${DB_URL}
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

<v-click>

Set environment variables in your deployment platform (Kubernetes secrets, Heroku config vars, AWS Parameter Store). Your credentials stay out of Git.

</v-click>

<!--
This is a security fundamentals reminder. Mention it briefly — students will encounter this pattern in every real project.
-->
