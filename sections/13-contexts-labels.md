---
layout: center
---
# Contexts & Labels

## Environment-Specific Migrations

---

# The Problem: Same Schema, Different Data

Your `tasks` table structure should be **identical** in dev, staging, and production.

But your **data** should differ:

<v-clicks>

- **Dev**: seed data with sample tasks and projects for testing
- **Staging**: production-like data, maybe a copy of prod
- **Production**: real data, no test records

</v-clicks>

<v-click>

How do you write a `seed-dev-tasks` changeset that only runs in dev?

**Answer: Contexts**

</v-click>

---

# Contexts Explained

A **context** is a tag you attach to a changeset. Liquibase only runs it when the active context matches.

```sql
--liquibase formatted sql

--changeset sunrise-team:007-seed-dev-data context:dev
INSERT INTO projects (name, created_at)
    VALUES ('Task Management System', CURRENT_TIMESTAMP);
INSERT INTO tasks (title, description, completed, project_id, created_at)
    VALUES ('Design login page UI',
            'Create wireframes and implement the login form',
            true, 1, CURRENT_TIMESTAMP);

--rollback DELETE FROM tasks WHERE title = 'Design login page UI';
--rollback DELETE FROM projects WHERE name = 'Task Management System';
```

<v-click>

This changeset only runs when `dev` context is active. In staging or production, it's silently skipped.

</v-click>

---

# Activating Contexts in application.properties

Tell Liquibase which context to activate based on the environment:

```properties
# application-dev.properties
spring.liquibase.contexts=dev

# application-staging.properties
spring.liquibase.contexts=staging

# application-prod.properties
spring.liquibase.contexts=prod
```

<v-click>

Then activate the profile when starting:

```bash
# macOS/Linux
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# Windows
mvnw.cmd spring-boot:run -Dspring-boot.run.profiles=dev

# Production (any OS)
java -jar app.jar --spring.profiles.active=prod
```

</v-click>

---

# Spring Profiles + Liquibase Contexts Together

The most common pattern: Spring profiles control which `application-{profile}.properties` loads.

```
src/main/resources/
├── application.properties          ← shared config (datasource, etc.)
├── application-dev.properties      ← spring.liquibase.contexts=dev
├── application-staging.properties  ← spring.liquibase.contexts=staging
└── application-prod.properties     ← spring.liquibase.contexts=prod
```

<v-click>

```properties
# application.properties (shared)
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.yaml
spring.liquibase.enabled=true
# No context here — set per environment
```

</v-click>

---

# Labels vs Contexts

Liquibase has two similar features: contexts and labels. They're used for different things:

<div class="grid grid-cols-2 gap-6 mt-4">

<div class="border rounded-lg p-4 border-blue-300 bg-blue-50 dark:bg-blue-950">

**Contexts** (environment-based)
- Set at **runtime** via properties
- Control **which changesets run**
- Use for: dev/staging/prod data
- Example: `context: dev`

</div>

<div class="border rounded-lg p-4 border-purple-300 bg-purple-50 dark:bg-purple-950">

**Labels** (deployment-based)
- Set at **deploy time** via CLI/Maven
- Flexible tagging — logical expressions
- Use for: feature flags, sprint tracking
- Example: `labels: sprint-5,feature-tasks`

</div>

</div>

<v-click>

For beginners: **use contexts**. Labels add complexity you don't need yet.

</v-click>

---

# Context Expressions

You can use boolean expressions in context values:

```yaml
# Run in dev OR staging
- changeSet:
    id: seed-non-prod-data
    context: "dev or staging"

# Run in dev AND when test flag is set
- changeSet:
    id: extreme-test-data
    context: "dev and test-data"

# Run everywhere except production
- changeSet:
    id: verbose-logging-trigger
    context: "!prod"
```

<v-click>

Keep it simple. The most common use case is `context: dev` for seed data and `context: prod` for production-only data transforms.

</v-click>

<!--
Don't overuse contexts. If a changeset should run everywhere, don't tag it with a context at all.
-->
