---
layout: center
---
# Rollback Strategies

## Undoing Migrations Safely

---

# Why Rollbacks Matter

Deployments fail. When they do, you need to get back to a known-good state quickly.

<v-clicks>

**Scenario:** You deploy a new version that adds an `assignee` column. The app throws exceptions because of a bug in the assignee validation logic. You need to roll back to the previous version.

**The schema problem:** The previous app version doesn't know about `assignee`. If the column exists but the old code doesn't expect it, you'll get errors.

**The solution:** Roll back the migration too — remove the `assignee` column when you revert the code.

</v-clicks>

---

# Automatic Rollback

Liquibase can automatically generate rollback SQL for many operations:

| Operation | Auto rollback |
|----------|--------------|
| `createTable` | `DROP TABLE` |
| `addColumn` | `DROP COLUMN` |
| `createIndex` | `DROP INDEX` |
| `addForeignKeyConstraint` | `DROP CONSTRAINT` |
| `insert` | `DELETE` |
| `renameColumn` | rename back |

<v-click>

These operations are **reversible** — Liquibase knows the inverse.

</v-click>

---

# Custom Rollback

Some operations can't be automatically reversed. You must write the rollback yourself:

```sql
--liquibase formatted sql

--changeset sunrise-team:009-mark-old-tasks-complete
UPDATE tasks
SET completed = true
WHERE created_at < NOW() - INTERVAL '90 days'
  AND completed = false;

--rollback UPDATE tasks SET completed = false WHERE created_at < NOW() - INTERVAL '90 days';
```

<v-click>

If a changeset has no automatic rollback AND no `--rollback` line, Liquibase will fail when you try to roll it back.

To explicitly mark a changeset as non-rollbackable, add an empty rollback comment:
```sql
--rollback empty
```

</v-click>

---

# Rollback Commands

Liquibase gives you several rollback strategies:

```bash
# Roll back the last N changesets
./mvnw liquibase:rollback -Dliquibase.rollbackCount=1       # macOS/Linux
mvnw.cmd liquibase:rollback -Dliquibase.rollbackCount=1     # Windows

# Roll back to a specific tag
./mvnw liquibase:rollback -Dliquibase.rollbackTag=v1.0      # macOS/Linux
mvnw.cmd liquibase:rollback -Dliquibase.rollbackTag=v1.0    # Windows

# Roll back to a date
./mvnw liquibase:rollbackToDate -Dliquibase.rollbackDate="2026-03-17 10:00:00"      # macOS/Linux
mvnw.cmd liquibase:rollbackToDate -Dliquibase.rollbackDate="2026-03-17 10:00:00"    # Windows
```

<v-click>

You can also preview the rollback SQL without executing it:

```bash
# Generate rollback SQL to a file (dry run)
./mvnw liquibase:rollbackSQL -Dliquibase.rollbackCount=1      # macOS/Linux
mvnw.cmd liquibase:rollbackSQL -Dliquibase.rollbackCount=1    # Windows
```

</v-click>

---

# Tagging Your Schema

Tags let you mark a point in time and roll back to it by name:

```bash
# After a successful release, tag the current schema state
./mvnw liquibase:tag -Dliquibase.tag=v1.0                     # macOS/Linux
mvnw.cmd liquibase:tag -Dliquibase.tag=v1.0                   # Windows

./mvnw liquibase:tag -Dliquibase.tag=release-2026-03-17       # macOS/Linux
mvnw.cmd liquibase:tag -Dliquibase.tag=release-2026-03-17     # Windows
```

<v-click>

Now if something goes wrong after deploying v2.0, you can roll back to the last tagged state:

```bash
./mvnw liquibase:rollback -Dliquibase.rollbackTag=v1.0      # macOS/Linux
mvnw.cmd liquibase:rollback -Dliquibase.rollbackTag=v1.0    # Windows
```

</v-click>

<v-click>

Best practice: tag before every production deployment. It costs nothing and saves you in emergencies.

</v-click>

---
zoom: 0.80
---

# Configuring Liquibase Maven Plugin

To use Maven rollback commands, add the plugin to `pom.xml`:

```xml
<build>
    <plugins>
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
    </plugins>
</build>
```

---

# Rollback Limitations

Not everything can be rolled back. Know these limitations:

<v-clicks>

**Data loss is permanent** — if a migration deleted data and you roll back the column, the data is gone

**`dropTable` can't be auto-rolled back** — write an explicit `rollback:` that recreates the table (but data is still lost)

**`modifyDataType`** — type changes that truncate data can't be automatically undone

**Raw SQL** — always needs an explicit `rollback:` block

</v-clicks>

<v-click>

This is why **additive changes are preferred**: add columns, don't delete them. Rename by adding a new column and migrating data over time.

</v-click>

---

# The Rollback Mental Model

Think of migrations like Git commits — you can revert, but you can't un-delete things.

<v-clicks>

**Forward migrations**: add features incrementally — low risk

**Rollbacks**: go back to a previous state — necessary when deployments fail

**Best strategy**: design migrations to be **additive** — add columns, add tables, add indexes. Removing things is always riskier.

**For deletes**: deprecate first (make nullable), then remove in a later release after confirming nothing uses it.

</v-clicks>

<!--
The real-world mindset: in production, you rarely rollback. You forward-fix. But you need rollback ability for the cases when you absolutely must.
-->
