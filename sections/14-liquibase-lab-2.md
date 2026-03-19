---
layout: center
---
# Lab 2: Evolve the Schema

## Add Indexes, Seed Dev Data, Test Contexts

<!--
15 minutes. Applying indexes, seed data, and contexts from the previous section.
-->

---

# What You're Doing

Your Task Flow API has 5 tables (`projects`, `users`, `roles`, `tasks`, `user_role`). In this lab you'll:

<v-clicks>

1. **Add indexes** on `tasks` — `completed`, `project_id`, and `created_at` columns
2. **Seed roles** — `VIEWER`, `USER`, `ADMIN`, `SUPER_ADMIN` on all environments, insert once
3. **Seed test data** — projects and tasks only for `dev` and `uat`
4. **Add a new column** — `assignee VARCHAR(50)` — to simulate schema evolution
5. **Test contexts** — verify test data doesn't run in prod mode

</v-clicks>

---

# Step 1 — Add Indexes on Tasks

Create `db/changelog/changes/006-add-task-indexes.sql`:

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

Then add it to `db.changelog-master.yaml`.

---

# Step 2 — Seed Roles (All Environments)

Create `db/changelog/changes/007-seed-roles.sql`:

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

No `context:` — roles are required in every environment. The precondition prevents duplicates if someone inserted them manually.

---

# Step 3 — Seed Test Data (Dev/UAT Only)

Create `db/changelog/changes/008-seed-dev-data.sql`:

```sql
--liquibase formatted sql

--changeset sunrise-team:008-seed-dev-data context:dev,uat
INSERT INTO projects (name, created_at) VALUES ('Task Management System', CURRENT_TIMESTAMP);
INSERT INTO projects (name, created_at) VALUES ('E-Commerce Platform', CURRENT_TIMESTAMP);
INSERT INTO tasks (title, completed, project_id, created_at)
    VALUES ('Design login page UI', true, 1, CURRENT_TIMESTAMP);
INSERT INTO tasks (title, completed, project_id, created_at)
    VALUES ('Implement authentication API', false, 1, CURRENT_TIMESTAMP);

--rollback DELETE FROM tasks WHERE title IN ('Design login page UI','Implement authentication API');
--rollback DELETE FROM projects WHERE name IN ('Task Management System','E-Commerce Platform');
```

Add both files to master changelog.

---

# Step 4 — Configure Contexts

Create or update `src/main/resources/application-dev.properties`:

```properties
spring.liquibase.contexts=dev
```

Create or update `src/main/resources/application-prod.properties`:

```properties
spring.liquibase.contexts=prod
```

The `context:dev,uat` on the changeset means it runs in either `dev` or `uat` — but not `prod`.

---

# Step 5 — Test Dev Context

Start the app with the `dev` profile:

```bash
# macOS/Linux
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# Windows
mvnw.cmd spring-boot:run -Dspring-boot.run.profiles=dev
```

Both changesets should run. Verify:

```sql
SELECT * FROM roles;       -- 4 rows: VIEWER, USER, ADMIN, SUPER_ADMIN
SELECT * FROM projects;    -- 2 rows: Task Management System, E-Commerce Platform
SELECT * FROM tasks;       -- 2 rows
```

---

# Step 6 — Test Prod Context

Stop the app. Clear test data and changelog entries:

```sql
DELETE FROM tasks;
DELETE FROM projects;
DELETE FROM roles;
DELETE FROM DATABASECHANGELOG WHERE id IN ('007-seed-roles', '008-seed-dev-data');
```

Start with prod profile:

```bash
# macOS/Linux
./mvnw spring-boot:run -Dspring-boot.run.profiles=prod

# Windows
mvnw.cmd spring-boot:run -Dspring-boot.run.profiles=prod
```

Then check:

```sql
SELECT * FROM roles;     -- 4 rows — roles run on all contexts
SELECT * FROM projects;  -- 0 rows — dev data skipped in prod
SELECT * FROM tasks;     -- 0 rows
```

---
zoom: 0.85
---

# Step 7 — Add assignee Column

Create `db/changelog/changes/008-add-assignee-to-tasks.sql`:

```sql
--liquibase formatted sql

--changeset sunrise-team:008-add-assignee-to-tasks
ALTER TABLE tasks ADD COLUMN assignee VARCHAR(50);

--rollback ALTER TABLE tasks DROP COLUMN assignee;
```

Add to master changelog and restart. The `assignee` column appears without dropping any data.

Now add it to `TaskModel.java`:

```java
private String assignee;
```

---

# ✓ Lab 2 Complete

<v-clicks>

- [ ] Indexes on `completed`, `project_id`, `created_at` columns are present in the DB
- [ ] Four roles present in both `dev` and `prod` profiles
- [ ] Projects and tasks appear in `dev` but not `prod`
- [ ] `assignee` column added to `tasks` table without data loss
- [ ] `TaskModel.java` updated to include the new `assignee` field

</v-clicks>

<!--
Break here — 10 minutes before rollback strategies.
-->
