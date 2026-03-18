---
layout: center
---
# Lab 3: Schema Change & Rollback

## Evolve and Revert Safely

<!--
20-minute lab. Students practice the full migration + rollback cycle. The most important lab in this module.
-->

---

# What You're Doing

You'll simulate a real deployment scenario:
1. Add a new migration (schema change)
2. Tag the current schema state
3. Roll back the migration
4. Verify the rollback worked

<v-clicks>

**The scenario:** The team wants to track time estimates for tasks. You add `estimated_hours` to the tasks table, deploy, and then the feature gets cut before it's used. You roll it back.

</v-clicks>

---

# Step 1 — Add the Liquibase Maven Plugin

Add to `pom.xml` inside `<build><plugins>`:

```xml
<plugin>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-maven-plugin</artifactId>
    <version>4.27.0</version>
    <configuration>
        <changeLogFile>
            src/main/resources/db/changelog/db.changelog-master.yaml
        </changeLogFile>
        <url>jdbc:postgresql://localhost:5432/taskflow_db</url>
        <username>postgres</username>
        <password>password</password>
        <driver>org.postgresql.Driver</driver>
    </configuration>
    <dependencies>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <version>42.7.3</version>
        </dependency>
    </dependencies>
</plugin>
```

---

# Step 2 — Tag the Current State

Before making changes, tag the current schema so we can roll back to it:

```bash
./mvnw liquibase:tag -Dliquibase.tag=before-estimated-hours
```

Verify in the database:

```sql
SELECT id, tag, dateexecuted
FROM DATABASECHANGELOG
ORDER BY dateexecuted DESC
LIMIT 3;
```

You should see a row with `tag = 'before-estimated-hours'`.

---

# Step 3 — Create the New Changeset

Create `db/changelog/changes/009-add-estimated-hours-column.sql`:

```sql
--liquibase formatted sql

--changeset sunrise-team:009-add-estimated-hours-column
ALTER TABLE tasks ADD COLUMN estimated_hours DECIMAL(5,1);

--rollback ALTER TABLE tasks DROP COLUMN estimated_hours;
```

Add it to `db.changelog-master.yaml`.

---

# Step 4 — Run the Migration

Start the app and verify the column was added:

```bash
./mvnw spring-boot:run
```

Then in the database:

```sql
-- Check the column exists
SELECT column_name, data_type, column_default
FROM information_schema.columns
WHERE table_name = 'tasks' AND column_name = 'estimated_hours';

-- Check DATABASECHANGELOG
SELECT id, author, dateexecuted
FROM DATABASECHANGELOG
ORDER BY dateexecuted;
```

---

# Step 5 — Roll Back

The feature was cut. Roll back to before estimated_hours was added:

```bash
# First, stop the app (migration runs on startup — we need direct DB access)
# Ctrl+C to stop

# Roll back to our tag
./mvnw liquibase:rollback -Dliquibase.rollbackTag=before-estimated-hours
```

<v-click>

You should see in the output:

```
Rolling Back Changeset: db/changelog/changes/009-add-estimated-hours-column.sql
  ::009-add-estimated-hours-column::sunrise-team
Liquibase: Rollback Successful
```

</v-click>

---
zoom: 0.85
---

# Step 6 — Verify Rollback

```sql
-- Column should be gone
SELECT column_name FROM information_schema.columns
WHERE table_name = 'tasks' AND column_name = 'estimated_hours';
-- Returns 0 rows

-- DATABASECHANGELOG entry should be removed
SELECT id FROM DATABASECHANGELOG WHERE id = '009-add-estimated-hours-column';
-- Returns 0 rows
```

Also verify the app starts successfully without the column:

```bash
./mvnw spring-boot:run
# Should start without errors
```

And tasks still have all their data:

```bash
curl -s http://localhost:8888/api/tasks | jq
```

---

# Step 7 — Preview Rollback SQL (Bonus)

Before rolling back in production, always preview what SQL will run:

```bash
./mvnw liquibase:rollbackSQL -Dliquibase.rollbackCount=1
```

Output:

```sql
-- *********************************************************************
-- Rollback 1 Change(s) Script
-- *********************************************************************
ALTER TABLE tasks DROP COLUMN estimated_hours;

DELETE FROM DATABASECHANGELOG
WHERE ID = '009-add-estimated-hours-column'
  AND AUTHOR = 'sunrise-team'
  AND FILENAME = 'db/changelog/changes/009-add-estimated-hours-column.sql';
```

<v-click>

Review this SQL before running the actual rollback in staging or production.

</v-click>

---

# ✓ Lab 3 Complete

<v-clicks>

- [ ] Maven Liquibase plugin configured in `pom.xml`
- [ ] Schema tagged with `before-estimated-hours`
- [ ] `estimated_hours` column added by new changeset
- [ ] Column successfully rolled back with `liquibase:rollback`
- [ ] App starts cleanly after rollback
- [ ] Existing task data unaffected

</v-clicks>

<v-click>

**You just performed a real production workflow** — tag, deploy, rollback, verify. This is exactly how migrations work on real teams.

</v-click>

<!--
Final break before the course recap. 10 minutes.
-->
