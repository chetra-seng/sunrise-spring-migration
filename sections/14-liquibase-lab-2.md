---
layout: center
---
# Lab 2: Evolve the Schema

## Add Indexes, Seed Dev Data, Test Contexts

<!--
15 minutes. Students apply what they just learned about indexes, seed data, and contexts.
-->

---

# What You're Doing

Your Task Flow API has 5 tables (`projects`, `users`, `roles`, `tasks`, `user_role`). In this lab you'll:

<v-clicks>

1. **Add indexes** on `tasks` — `completed`, `project_id`, and `created_at` columns
2. **Seed dev data** — 2 projects and 3 tasks that only appear in development
3. **Add a new column** — `assignee VARCHAR(50)` — to simulate schema evolution
4. **Test contexts** — verify seed data doesn't run in prod mode

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
zoom: 0.85
---

# Step 2 — Create Dev Seed Data

Create `db/changelog/changes/007-seed-dev-data.sql`:

```sql
--liquibase formatted sql

--changeset sunrise-team:007-seed-dev-data context:dev
INSERT INTO projects (name, created_at)
    VALUES ('Task Management System', CURRENT_TIMESTAMP);
INSERT INTO projects (name, created_at)
    VALUES ('E-Commerce Platform', CURRENT_TIMESTAMP);
```

---
zoom: 0.85
---

# Step 2 — Seed Tasks (continued)

Add the task inserts to the same `007-seed-dev-data.sql` file (after the projects):

```sql
INSERT INTO tasks (title, description, completed, project_id, created_at)
    VALUES ('Design login page UI',
            'Create wireframes and implement the login form',
            true, 1, CURRENT_TIMESTAMP);
INSERT INTO tasks (title, description, completed, project_id, created_at)
    VALUES ('Implement authentication API',
            'JWT-based login and register endpoints',
            false, 1, CURRENT_TIMESTAMP);
INSERT INTO tasks (title, description, completed, project_id, created_at)
    VALUES ('Shopping cart integration',
            'Add to cart, update quantity, checkout flow',
            false, 2, CURRENT_TIMESTAMP);

--rollback DELETE FROM tasks WHERE title IN ('Design login page UI','Implement authentication API','Shopping cart integration');
--rollback DELETE FROM projects WHERE name IN ('Task Management System','E-Commerce Platform');
```

Add to master changelog.

---

# Step 3 — Configure Dev Context

Create `src/main/resources/application-dev.properties`:

```properties
spring.liquibase.contexts=dev
```

Create `src/main/resources/application-prod.properties`:

```properties
spring.liquibase.contexts=prod
```

---

# Step 4 — Test Dev Context

Start the app with the `dev` profile:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev
```

Then query (you'll need a JWT token — get one via `/api/auth/login`):

```bash
curl -s http://localhost:8888/api/tasks \
  -H "Authorization: Bearer $TOKEN" | jq
```

You should see three tasks (Design login page UI, Implement authentication API, Shopping cart integration).

---

# Step 5 — Test Prod Context

Stop the app. Drop the seeded data (but not the tables):

```sql
DELETE FROM tasks;
DELETE FROM projects;
DELETE FROM DATABASECHANGELOG WHERE id = '007-seed-dev-data';
```

Start with prod profile:

```bash
./mvnw spring-boot:run -Dspring-boot.run.profiles=prod
```

```bash
curl -s http://localhost:8888/api/tasks \
  -H "Authorization: Bearer $TOKEN" | jq
# Should return []  — no seed data
```

---
zoom: 0.85
---

# Step 6 — Add assignee Column

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
- [ ] Three tasks appear when running with `dev` profile
- [ ] No tasks inserted when running with `prod` profile
- [ ] `assignee` column added to `tasks` table without data loss
- [ ] `TaskModel.java` updated to include the new `assignee` field

</v-clicks>

<!--
Break here — 10 minutes before rollback strategies.
-->
