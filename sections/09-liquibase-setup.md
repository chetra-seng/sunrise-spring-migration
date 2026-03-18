---
layout: center
---
# Liquibase Setup

## Adding Liquibase to Your Spring Boot Project

---

# Step 1 — Add the Dependency

```xml
<dependencies>
    <!-- existing dependencies ... -->

    <!-- Liquibase for database migrations -->
    <dependency>
        <groupId>org.liquibase</groupId>
        <artifactId>liquibase-core</artifactId>
    </dependency>
</dependencies>
```

<v-click>

No version needed — Spring Boot's dependency management (`spring-boot-starter-parent`) provides a compatible Liquibase version automatically.

</v-click>

---

# Step 2 — Update application.properties

Make two changes:

````md magic-move
```properties
# Before: Hibernate silently modifies the schema — no history, no control
spring.jpa.hibernate.ddl-auto=update
```

```properties
# After: Liquibase manages the schema instead
spring.jpa.hibernate.ddl-auto=validate

# Liquibase configuration
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.yaml
spring.liquibase.enabled=true
```
````

<v-click>

`validate` means: "Check that the actual DB matches my `@Entity` classes, but don't change anything." Liquibase handles the changes now.

</v-click>

---

# Step 3 — Directory Structure

Create this directory structure under `src/main/resources`:

```
src/
└── main/
    └── resources/
        ├── application.properties
        └── db/
            └── changelog/
                ├── db.changelog-master.yaml    ← entry point (YAML index)
                └── changes/
                    ├── 001-create-projects-table.sql
                    ├── 002-create-users-table.sql
                    ├── 003-create-roles-table.sql
                    ├── 004-create-tasks-table.sql
                    └── 005-create-user-role-table.sql
```

<v-click>

The `db/changelog/changes/` pattern keeps individual changesets organized. The master file includes them in order.

</v-click>

---

# Step 4 — Create the Master Changelog

Create `src/main/resources/db/changelog/db.changelog-master.yaml`:

```yaml
databaseChangeLog:
  - include:
      file: db/changelog/changes/001-create-projects-table.sql
      relativeToChangelogFile: false
```

<v-click>

Start with just one include. You'll add more as you write individual changesets.

</v-click>

<v-click>

The `relativeToChangelogFile: false` means paths are relative to the classpath root (the `resources/` directory). This is the recommended setting.

</v-click>

---
zoom: 0.85
---

# Step 5 — Disable ddl-auto Warnings

You may see a warning when using Liquibase with `ddl-auto=validate`:

```
HHH90000025: DDL-auto is 'validate' but no DDL validation logic is configured
```

Add this to `application.properties` to suppress it:

```properties
# Disable Hibernate schema validation warnings when using Liquibase
spring.jpa.properties.hibernate.hbm2ddl.auto=none
```

Or simply use `ddl-auto=none` which tells Hibernate to stay out of schema management entirely:

```properties
spring.jpa.hibernate.ddl-auto=none
```

<v-click>

Use `none` in production — your Liquibase changelogs are the single source of truth for the schema.

Use `validate` in development to catch mismatches between your `@Entity` classes and the actual database.

</v-click>

---

# The Full application.properties

```properties
server.port=8888

# PostgreSQL
spring.datasource.url=jdbc:postgresql://localhost:5432/taskflow_db
spring.datasource.username=postgres
spring.datasource.password=password
spring.datasource.driver-class-name=org.postgresql.Driver

# JPA — let Liquibase manage the schema
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Liquibase
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.yaml
spring.liquibase.enabled=true
```

---

# What Happens on First Run

When you start the app with Liquibase configured and an empty database:

<v-clicks>

1. Liquibase connects to the database
2. It creates two control tables: `DATABASECHANGELOG` and `DATABASECHANGELOGLOCK`
3. It reads `db.changelog-master.yaml`
4. It executes each included changeset in order
5. Each executed changeset is recorded in `DATABASECHANGELOG`
6. Spring Boot completes startup

</v-clicks>

<v-click>

The `DATABASECHANGELOGLOCK` table prevents two instances from running migrations simultaneously — important for multi-instance deployments.

</v-click>

---

# Two Paths: Fresh vs Existing Project

<div class="grid grid-cols-2 gap-6 mt-4">

<div class="border rounded-lg p-4 border-blue-300 bg-blue-50 dark:bg-blue-950">

**Fresh database** ← labs use this
- Write changelogs describing your schema
- Start the app — Liquibase creates everything
- Clean mental model: Liquibase owns the schema from day one

</div>

<div class="border rounded-lg p-4 border-amber-300 bg-amber-50 dark:bg-amber-950">

**Existing project** ← real world
- Tables already exist (from `ddl-auto=update`)
- Write changelogs to match the *current* schema
- Run `changelogSync` — marks them as done without running them
- Future changes go through Liquibase from here

</div>

</div>

<v-click>

In the labs we use a fresh database so you can see Liquibase work without worrying about existing data. In a real project, you'd never drop tables — you'd use `changelogSync`.

</v-click>

---

# Existing Project: changelogSync

When adding Liquibase to a project that already has tables:

```bash
# Step 1: write changelogs that match your current schema exactly
# Step 2: tell Liquibase "these are already done — don't run them"
./mvnw liquibase:changelogSync

# Step 3: from now on, all schema changes go through new changelogs
```

<v-click>

`changelogSync` records every changeset in `DATABASECHANGELOG` as if it ran — but touches nothing in the actual database. The existing schema stays exactly as-is.

</v-click>

<v-click>

After `changelogSync`, your next migration runs normally. Liquibase is now in control.

```bash
# Verify — all changesets show as executed:
./mvnw liquibase:status
```

</v-click>

<!--
Contrast with dropping tables: changelogSync is safe, zero-downtime, and the professional approach. Never drop production tables just to add Liquibase.
-->

