---
layout: center
---
# Course Recap

## What You've Built and Learned

---

# The Full Journey

Five modules. One evolving Task Flow API. Here's where you are now:

<div class="mt-4 text-sm">

```
Module 1: Java + Spring Boot Basics
  └─ TaskController, TaskServiceImpl, DTOs, MapStruct mappers

Module 2: REST APIs & Error Handling
  └─ Proper HTTP status codes, @ExceptionHandler, validation

Module 3: Spring Data JPA + PostgreSQL
  └─ @Entity, JpaRepository, real database persistence

Module 4: Spring Security
  └─ JWT auth, role-based access, protected endpoints

Documentation + Migrations  ← YOU ARE HERE
  └─ Swagger UI, Liquibase changelogs, rollback strategies
```

</div>

<v-click>

You have a **production-ready** API — not just working code, but documented, secure, and safely deployable.

</v-click>

---

# What You Added

<div class="grid grid-cols-2 gap-6 mt-4">

<div>

**Swagger / OpenAPI**
- `springdoc-openapi-starter-webmvc-ui` dependency
- `OpenApiConfig` bean — title, description, version
- `@Tag` on `TaskController`
- `@Operation`, `@ApiResponse` on each endpoint
- `@Schema` on `TaskRequest` and `TaskResponse`
- Live docs at `/swagger-ui.html`

</div>

<div>

**Liquibase**
- `liquibase-core` dependency
- `ddl-auto=validate` (Liquibase manages schema)
- `db.changelog-master.yaml` entry point
- `001-create-projects-table.yaml`
- `002-create-users-table.yaml`
- `003-create-roles-table.yaml`
- `004-create-tasks-table.yaml`
- `005-create-user-role-table.yaml`
- Rollback with Maven plugin + tags

</div>

</div>

---

# Architecture: Where Everything Lives

<div class="grid grid-cols-2 gap-6 mt-4 text-sm">

<div>

```
src/main/
└── java/com/chetraseng/sunrise_task_flow_api/
    ├── config/
    │   ├── OpenApiConfig.java         ← NEW
    │   └── SecurityConfig.java
    ├── controllers/
    │   ├── TaskController.java        ← annotated
    │   ├── AuthController.java
    │   └── RoleController.java
    ├── dto/
    │   ├── TaskRequest.java           ← @Schema
    │   └── TaskResponse.java          ← @Schema
    ├── model/
    │   ├── TaskModel.java
    │   ├── ProjectModel.java
    │   ├── UserModel.java
    │   ├── RoleModel.java
    │   └── UserRoleModel.java
    ├── mapper/
    │   └── TaskMapper.java
    ├── services/
    │   └── TaskServiceImpl.java
    └── exception/
        └── ErrorResponse.java
```

</div>

<div>

```
src/main/resources/
├── application.properties
├── application-dev.properties         ← updated
├── application-prod.properties        ← NEW
└── db/
    └── changelog/
        ├── db.changelog-master.yaml   ← NEW
        └── changes/
            ├── 001-create-projects-table.yaml  ← NEW
            ├── 002-create-users-table.yaml     ← NEW
            ├── 003-create-roles-table.yaml     ← NEW
            ├── 004-create-tasks-table.yaml     ← NEW
            └── 005-create-user-role-table.yaml ← NEW
```

</div>

</div>

---

# Key Concepts to Remember

<v-clicks>

**OpenAPI/Swagger:** code-first documentation — annotate your code, get a UI for free

**The golden rule of Liquibase:** never modify a changeset that has already been applied

**ddl-auto in production:** always `none` or `validate` — never `update`, `create`, or `create-drop`

**Contexts:** environment-specific changesets — use `context: dev` for seed data

**Tags:** mark a known-good schema state before every production deployment

**Rollbacks:** write explicit `rollback:` blocks for any non-reversible changesets

</v-clicks>

---

# Resources

**Swagger / OpenAPI:**
- SpringDoc OpenAPI: <a href="https://springdoc.org/" target="_blank">springdoc.org</a>
- Swagger UI: <a href="https://swagger.io/tools/swagger-ui/" target="_blank">swagger.io/tools/swagger-ui</a>
- OpenAPI Specification: <a href="https://spec.openapis.org/oas/v3.1.0" target="_blank">spec.openapis.org</a>

**Liquibase:**
- Official docs: <a href="https://docs.liquibase.com/" target="_blank">docs.liquibase.com</a>
- Change types reference: <a href="https://docs.liquibase.com/change-types/home.html" target="_blank">docs.liquibase.com/change-types</a>
- Best practices: <a href="https://docs.liquibase.com/workflows/liquibase-community/home.html" target="_blank">docs.liquibase.com/workflows</a>

**Spring Boot:**
- Spring Boot + Liquibase: <a href="https://docs.spring.io/spring-boot/how-to/data-initialization.html" target="_blank">docs.spring.io — data initialization</a>
