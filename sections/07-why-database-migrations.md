---
layout: center
---
# Why Database Migrations?

## The Problem with ddl-auto

---

# You've Been Using ddl-auto=update

Since Module 3, your `application-dev.properties` has had:

```properties
spring.jpa.hibernate.ddl-auto=update
```

It's been working for development. But let's think about what it actually does.

<v-click>

**`ddl-auto=update` means:** "When the app starts, compare my `@Entity` classes to the actual database schema and add any missing tables or columns — but never drop anything."

</v-click>

<v-click>

That sounds safe. What could go wrong?

</v-click>

---

# The Problems with ddl-auto=update

With `ddl-auto=update`, Hibernate silently modifies your schema on every restart:

<v-clicks>

**No audit trail.** You can't tell what changed, when, or why.

**It only adds, never removes.** Dropped columns in your entity leave orphaned columns in the DB.

**Renaming a column breaks it.** Hibernate adds the new name but leaves the old column with its data.

**Works differently across environments.** Dev schema drifts from staging and production.

</v-clicks>

<!--
This scenario resonates with students. Ask: "Has anyone lost dev data because of this?"
-->

---

# The Problems Scale in Teams

**Scenario: Two developers, one shared database.**

<v-clicks>

**Dev A** adds an `assignee` field to `TaskModel`, runs the app → table is re-created with `assignee` column

**Dev B** adds an `estimated_hours` field to `TaskModel`, runs the app → table is re-created with `estimated_hours`

They both `git push`. A third developer pulls and runs.

**The database gets whichever schema ran last** — there's no record of the history.

Then someone needs to add a column to staging or production — there's no migration script to run.

</v-clicks>

---

# ddl-auto Cannot Handle Everything

There are schema changes that `ddl-auto` simply can't do safely:

<v-clicks>

**Renaming a column** → it adds a new column and leaves the old one (data stranded)

**Changing a column type** → it may fail or silently truncate data

**Dropping a column** → only happens with `create-drop`, which destroys all data

**Adding a NOT NULL column to existing table** → fails because existing rows have no value

**Seeding initial data** → `ddl-auto` manages structure only, not data

</v-clicks>

<v-click>

These are exactly the changes you need to make as a real application evolves.

</v-click>

---

# The Production Problem

In production, you can **never** use `ddl-auto=update`, `create`, or `create-drop`.

<div class="grid grid-cols-2 gap-6 mt-4">

<div class="border rounded-lg p-4 border-red-300 bg-red-50 dark:bg-red-950">

**What ddl-auto does in production:**
- Drops and recreates tables → loses all data
- Adds columns without defaults → breaks existing queries
- Can't be rolled back if something goes wrong
- No audit trail — who changed what and when?
- Operations team can't review or approve

</div>

<div class="border rounded-lg p-4 border-green-300 bg-green-50 dark:bg-green-950">

**What you need in production:**
- Explicit, reviewed SQL scripts
- Tracked history of every change
- Ability to roll back if a deployment fails
- Same scripts in dev, staging, and prod
- Automated execution on deploy

</div>

</div>

---

# The Solution: Database Migrations

A migration tool gives you version control for your database schema.

<v-clicks>

**The idea:** Instead of letting Hibernate guess how to evolve the schema, you write explicit migration scripts.

**Each script** describes exactly one schema change — "create table projects" or "add column assignee to tasks".

**The tool tracks** which scripts have run. It only runs new ones.

**The scripts live in Git** alongside your code — reviewed, approved, and deployed the same way.

</v-clicks>

<v-click>

This is how every production application manages its schema. The two most popular tools for Spring Boot are **Flyway** and **Liquibase**.

</v-click>

---

# Flyway vs Liquibase

<div class="grid grid-cols-2 gap-6 mt-4">

<div class="border rounded-lg p-4 border-blue-300 bg-blue-50 dark:bg-blue-950">

**Flyway**
- SQL-only migrations
- Simple file naming convention (`V1__init.sql`)
- Easier for SQL-first teams
- Less flexible for environments
- Free tier covers most use cases

</div>

<div class="border rounded-lg p-4 border-green-300 bg-green-50 dark:bg-green-950">

**Liquibase**
- XML, YAML, JSON, or SQL changelogs
- More powerful: contexts, labels, preconditions
- Environment-specific migrations
- Better enterprise tooling
- Open source core is free

</div>

</div>

<v-click>

We'll use **Liquibase** because it teaches more concepts and is more common in enterprise Spring Boot projects.

</v-click>

<!--
Both are good choices. Tell students that knowing one makes the other easy to pick up.
-->
