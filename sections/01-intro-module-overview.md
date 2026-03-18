# Where We Are

You've built a Task Flow API with Spring Boot, JPA, and PostgreSQL. It works — but it has two gaps:

<v-clicks>

**Gap 1: Nobody knows how to use it.**
- No documentation
- Other developers have to read your code to understand the endpoints
- Frontend team can't work without endpoint specs

**Gap 2: Schema changes are risky.**
- `ddl-auto=update` can't safely handle column renames, type changes, or production deploys
- Team members get different schemas
- Deploying to production is nerve-wracking

</v-clicks>

<!--
Pause after each gap. Ask: "Has anyone experienced this frustration?" — builds motivation.
-->

---

# This Fixes Both

<div class="grid grid-cols-2 gap-6 mt-4">

<div>

### Part 1 — Swagger (45 min)
Document your API automatically from annotations.

- Why documentation matters
- Add `springdoc-openapi`
- `@Tag`, `@Operation`, `@Parameter`, `@ApiResponse`
- `@Schema` on DTOs
- **Lab**: Document the full Task Flow API

</div>

<div>

### Part 2 — Liquibase (2h 30min)
Version-control your database schema.

- Why `ddl-auto` isn't enough
- Changelogs & changesets
- YAML and XML syntax
- Contexts, labels, rollbacks
- **Labs**: Set up, evolve, and roll back

</div>

</div>

<!--
~5 minutes total for intro. Keep it moving — students want to see real code.
-->
