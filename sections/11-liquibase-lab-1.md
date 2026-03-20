---
layout: center
---
# Lab 1: Set Up Liquibase

## Migrate the Task Flow API to Liquibase

<!--
This is the foundational lab. 25 minutes. The biggest hurdle is YAML indentation.
-->

---

# What You're Doing

Your Task Flow API uses `ddl-auto=update`. You're going to replace it with Liquibase migrations.

<v-clicks>

**Lab setup:** We're using a **fresh empty database** — so you'll see Liquibase create all tables from scratch. This is the cleanest way to learn how changelogs work.

> In a real existing project you'd use `changelogSync` instead of dropping tables — that's covered in the setup section.

**What changes:**
- `pom.xml` — add `spring-boot-starter-liquibase`
- `application-dev.properties` — switch from `ddl-auto=update` to `ddl-auto=validate`
- New files: `db.changelog-master.yaml` (YAML index) and five `.sql` changeset files (one per table)

**What stays the same:**
- `TaskModel.java` — untouched
- `TaskController.java` — untouched
- `TaskServiceImpl.java` — untouched

</v-clicks>

---

# Step 1 — Add Liquibase Dependency

Open `pom.xml` and add inside `<dependencies>`:

```xml
<!-- Liquibase database migrations -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-liquibase</artifactId>
</dependency>
```

No version tag — Spring Boot manages it.

---

# Step 2 — Update application-dev.properties

<Transform :scale="0.9">

```properties
server.port=8888

# PostgreSQL connection
spring.datasource.url=jdbc:postgresql://localhost:5432/taskflow_db
spring.datasource.username=postgres
spring.datasource.password=password

# JPA — Liquibase manages the schema now
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

# Liquibase
spring.liquibase.change-log=classpath:db/changelog/db.changelog-master.yaml
spring.liquibase.enabled=true

# Keep JWT config as-is
jwt.secret=${JWT_SECRET}
jwt.expiration=86400000
jwt.refresh-expiration=604800000
```

<v-click>

`validate` means Hibernate will check that your DB matches your `@Entity` classes but won't modify anything — Liquibase owns all schema changes now.

</v-click>

</Transform>


---

# Step 3 — Start With a Fresh Database

For this lab, drop all existing tables so Liquibase can create them fresh:

```sql
DROP TABLE IF EXISTS user_role;
DROP TABLE IF EXISTS tasks;
DROP TABLE IF EXISTS users;
DROP TABLE IF EXISTS roles;
DROP TABLE IF EXISTS projects;
```

Drop in dependency order — child tables first (foreign key constraints).

<div class="mt-3 p-3 border border-amber-400 rounded bg-amber-50 dark:bg-amber-950 text-sm">

**Real project?** Don't drop tables. Instead, write changelogs to match your existing schema, then run `./mvnw liquibase:changelogSync` to mark them as done without touching the database.

</div>

---

# Step 4 — Create the Directory Structure

```bash
mkdir -p src/main/resources/db/changelog/changes
```

Your `resources` folder should look like:
```
resources/
├── application.properties
└── db/
    └── changelog/
        ├── db.changelog-master.yaml        ← to create (YAML index)
        └── changes/                        ← to create .sql changesets here
```

---

# Step 5 — Create the Master Changelog

Create `src/main/resources/db/changelog/db.changelog-master.yaml`:

```yaml
databaseChangeLog:
  - include:
      file: db/changelog/changes/001-create-projects-table.sql
      relativeToChangelogFile: false
  - include:
      file: db/changelog/changes/002-create-users-table.sql
      relativeToChangelogFile: false
  - include:
      file: db/changelog/changes/003-create-roles-table.sql
      relativeToChangelogFile: false
  - include:
      file: db/changelog/changes/004-create-tasks-table.sql
      relativeToChangelogFile: false
  - include:
      file: db/changelog/changes/005-create-user-role-table.sql
      relativeToChangelogFile: false
```

<!--
Warn about YAML indentation — 2 spaces, never tabs. This trips up most beginners.
-->

---
zoom: 0.85
---

# Step 6 — Create 001-create-projects-table.sql

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

---
zoom: 0.85
---

# Step 6 — Create 002-create-users-table.sql

```sql
--liquibase formatted sql

--changeset sunrise-team:002-create-users-table
CREATE TABLE users (
    id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    email      VARCHAR(255) NOT NULL UNIQUE,
    first_name VARCHAR(150),
    last_name  VARCHAR(150),
    password   VARCHAR(255),
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

--rollback DROP TABLE users;
```

---
zoom: 0.85
---

# Step 6 — Create 003-create-roles-table.sql

```sql
--liquibase formatted sql

--changeset sunrise-team:003-create-roles-table
CREATE TABLE roles (
    id   BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE
);

--rollback DROP TABLE roles;
```

---
zoom: 0.85
---

# Step 6 — Create 004-create-tasks-table.sql

```sql
--liquibase formatted sql

--changeset sunrise-team:004-create-tasks-table
CREATE TABLE tasks (
    id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    title       VARCHAR(100) NOT NULL,
    description TEXT,
    completed   BOOLEAN DEFAULT false,
    created_at  TIMESTAMP,
    project_id  BIGINT REFERENCES projects(id)
);

--rollback DROP TABLE tasks;
```

---
zoom: 0.85
---

# Step 6 — Create 005-create-user-role-table.sql

```sql
--liquibase formatted sql

--changeset sunrise-team:005-create-user-role-table
CREATE TABLE user_role (
    id         BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    created_at TIMESTAMP,
    user_id    BIGINT NOT NULL REFERENCES users(id),
    role_id    BIGINT NOT NULL REFERENCES roles(id)
);

--rollback DROP TABLE user_role;
```

---

# Step 7 — Start the Application

```bash
# macOS/Linux
./mvnw spring-boot:run

# Windows
mvnw.cmd spring-boot:run
```

Look for these lines in the startup log:

```
Liquibase: Successfully acquired change log lock
Running Changeset: ...001-create-projects-table...
Running Changeset: ...002-create-users-table...
Running Changeset: ...003-create-roles-table...
Running Changeset: ...004-create-tasks-table...
Running Changeset: ...005-create-user-role-table...
Liquibase: Successfully released change log lock
```

---
zoom: 0.85
---

# Step 8 — Verify in the Database

Connect to PostgreSQL and check:

```sql
-- Check Liquibase's tracking table (should show 5 entries)
SELECT id, author, dateexecuted
FROM DATABASECHANGELOG;

-- Verify the tasks table was created with correct columns
\d tasks

-- Or query information_schema
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'tasks'
ORDER BY ordinal_position;
```

Expected result:
```
              id              |   author    | dateexecuted
------------------------------+-------------+--------------
001-create-projects-table     | sunrise-team | 2026-03-17
002-create-users-table        | sunrise-team | 2026-03-17
003-create-roles-table        | sunrise-team | 2026-03-17
004-create-tasks-table        | sunrise-team | 2026-03-17
005-create-user-role-table    | sunrise-team | 2026-03-17
```

---

# Step 9 — Test the API

The API requires a JWT token. First register and login to get one:

```bash
# Register a new user
curl -s -X POST http://localhost:8888/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"firstName":"Alice","lastName":"Dev","email":"alice@dev.com","password":"password123"}' | jq

# Login to get a token
TOKEN=$(curl -s -X POST http://localhost:8888/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"alice@dev.com","password":"password123"}' | jq -r '.accessToken')

# Create a task
curl -s -X POST http://localhost:8888/api/tasks \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"title":"Implement login page","description":"Build the login form UI"}' | jq

# Get all tasks
curl -s http://localhost:8888/api/tasks \
  -H "Authorization: Bearer $TOKEN" | jq
```

---

# Troubleshooting

| Error | Likely Cause | Fix |
|-------|-------------|-----|
| `ChangeLogParseException` | YAML indentation error | Check 2-space indent, no tabs |
| `Validation failed for changeset` | MD5 mismatch | Don't edit applied changesets — write a new one |
| `Table 'tasks' already exists` | Dropped incorrectly | Drop all tables in order, then restart |
| `Could not find changelog file` | Wrong path in properties | Check classpath path matches file location |
| `Connection refused` | PostgreSQL not running | Start postgres or Docker |

---

# ✓ Lab 1 Complete

**Your database schema is now version-controlled.**

<v-click>

Every future schema change will be a new `.sql` file in `changes/`. Your team can review them in Git before they run. New team members get the full schema history automatically.

</v-click>

<!--
5-minute break here, then continue to advanced changelogs.
-->
